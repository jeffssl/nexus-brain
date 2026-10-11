---
fecha: 2026-10-04
tags: [log, base-de-datos, auditoria, data-first, postgresql, decision]
tipo: log
---

# 🧠 Diario: Auditoría de `Nexus.dbml` y decisión Data-First

> [!success] Hito
> Se auditó el modelo completo, se decidió **abandonar DBML como fuente de verdad** y adoptar **Data-First (SQL versionado)**. Roadmap F0–F9 aprobado para revisión.

## 1. Auditoría (`Nexus.dbml`, 55 tablas)
- Criterios: [[Reglas_Base_de_Datos]] (health-tech-db-architect + postgres-pro).
- Validación: `npx -y -p @dbml/cli dbml2sql brain/01_Architecture/Nexus.dbml --postgres -o nexus.sql` → compila (55 tablas, 137 FKs, solo 28 índices).
- Hallazgos críticos verificados en el SQL exportado:
  - `default: 'now()'`/`'CURRENT_DATE'` (comillas simples) se exportan como **literales de texto**; deben ir con backticks.
  - `GENERATED ALWAYS AS IDENTITY` solo estaba en `note:` → ~35 PK sin generador.
  - `uuid_generate_v7()` **no existe en pgcrypto** (requiere PG18 `uuidv7()`, `pg_uuidv7` o `Guid.CreateVersion7()`).
  - Unicidades globales (documento, appointment_number, invoice_number…) rompen multi-tenancy y filtran existencia entre tenants.
  - Sin FKs compuestas `(tenant_id, id)`: una cita de A puede apuntar a un paciente de B.
  - `audit_logs` y `appointment_archives` con PHI sin `tenant_id` → sin RLS posible.
  - `EXCLUDE USING gist` solo en `note`; los índices únicos no evitan solapamientos.
  - `slots.max_capacity` > 1 contradice `appointments.slot_id` único.
  - Laboratorio no modelable (`practitioner_id NOT NULL`, servicios sin tipo).
- Drift con el código: `Organization` (UUID v4 con `Guid.NewGuid()`, columnas `NOT NULL` en la migración `Initial` vs. nullables en el DBML, faltan auditoría y FKs; namespace `Nexus.Infraestructure` con typo).
- Detalle completo: [[Auditoria_Nexus_DBML_2026-10-04]].

## 2. Decisión: ¿seguir o rehacer?
- Se recomendó **no empezar de cero**: el dominio es sólido; los fallos son de implementación. `Nexus_Learning.dbml` pasa a ser el modelo de aprendizaje y `Nexus.dbml` solo referencia.
- `Nexus_Learning.dbml` repite 2 errores (`'now()'` y `unique` globales).

## 3. Decisión: Data-First
> [!info] Justificación técnica
> DBML/EF no expresan `EXCLUDE USING gist`, RLS, `CHECK`, índices parciales reales, particionado ni `IDENTITY`. En un sistema de salud multi-tenant con concurrencia, esas garantías deben vivir en PostgreSQL.
- **Fuente de verdad:** `src/Database/migrations/*.sql` (DbUp, forward-only).
- **EF Core:** solo mapeo, sin migraciones (se descarta la migración `Initial`).
- **DBML:** artefacto **generado**: `npx -p @dbml/cli db2dbml postgres "<conn>" -o brain/01_Architecture/Nexus.dbml`.
- Comparativa Code-First vs Data-First registrada en el chat; criterio: usar Code-First para CRUD/MVP y Data-First cuando se requieran RLS, EXCLUDE, particionado o control fino de locks.

## 4. Roadmap F0–F9
[[Roadmap_Data_First]]: F0 decisiones · F1 plataforma BD (roles, RLS, runner, Testcontainers) · F2 tenancy+identidad · F3 catálogos · F4 actores · F5 agendamiento (EXCLUDE, concurrencia) · F6 laboratorio · F7 seguros/facturación · F8 auditoría/outbox/particiones · F9 hardening. Estimado 31–43 días.

> [!warning] Decisiones pendientes (F0)
> PG18 vs anterior · médico global (`PractitionerRole`) vs por tenant · catálogo de servicios híbrido · 1 slot = 1 cita · SRI fuera de alcance · DbUp.

## Próximos pasos
1. Confirmar D1–D9 y ejecutar F0 (ADRs, `src/Database/`, docker-compose, actualizar `AGENTS.md` y regla E2E).
2. Corregir `Nexus_Learning.dbml` mientras se migra a SQL.

---
## 🚀 2026-10-11 — Ejecución de la Fase 0

> [!success] F0 completada
> Roadmap aprobado; decisiones D1–D9 aceptadas como ADR.

- **Hallazgo:** `src/` está en `.gitignore` del repo raíz → la BD **no** puede vivir en `src/Database`. Se creó `infra/database/` (versionada).
- Creado: `infra/database/{migrations,seeds,tests}`, `docker-compose.yml` (postgres:18, volumen en `/var/lib/postgresql` según cambio de PG18), `.env.example`, `README.md`, `conventions.md`; `.env` agregado a `.gitignore`.
- ADRs: [[ADR-D1_PostgreSQL_18]] … [[ADR-D9_Soft_Delete_Unificado]] (índice en [[MOC_Arquitectura_Base_de_Datos]]).
- Reglas: `.agents/AGENTS.md` (sección 7 Data-First) y regla E2E actualizadas: la fuente de verdad pasa a `infra/database/migrations/*.sql`.
- **Siguiente:** F1 (extensiones, schemas, roles, funciones, RLS template, runner DbUp, Testcontainers).
