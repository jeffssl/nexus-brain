---
title: "Auditoría de Nexus.dbml (2026-10-04)"
fecha: 2026-10-04
tipo: auditoria
estado: activo
tags: [arquitectura, base-de-datos, auditoria, postgresql, rls, hipaa]
---

> [!info] Contexto
> Auditoría que motivó la decisión de pasar a Data-First. Ver [[Roadmap_Data_First]] y [[2026-10-04_Auditoria_DBML_y_Decision_Data_First]]. Modelo auditado: [[Esquema_Nexus]], [[Reglas_Base_de_Datos]].

# Auditoría de `Nexus.dbml`

**Archivo:** [Nexus.dbml](file:///Users/jeff/Documents/Agendamiento-clinicas-laboratorios/brain/01_Architecture/Nexus.dbml)
**Criterios:** `health-tech-db-architect` (FHIR, RLS, concurrencia, HIPAA/LOPDP, auditoría) + `postgres-pro`
**Validación:** el DBML compila (`@dbml/cli dbml2sql`): 55 tablas, 137 FKs, **solo 28 índices**.

| Severidad | Cantidad |
|---|---|
| 🔴 Crítico (bug real / fuga de datos) | 8 |
| 🟠 Alto (diseño que bloquea casos de uso) | 9 |
| 🟡 Medio (integridad / rendimiento) | 10 |
| ⚪ Bajo (consistencia / nomenclatura) | 6 |

---

## 🔴 Críticos

### C1. El DDL generado está roto (defaults e identity)
Verificado en el SQL exportado:
- `default: 'now()'` → `DEFAULT 'now()'` (**literal string**, no función). 7 columnas.
- `default: 'CURRENT_DATE'` en `billing.invoices.issue_date` → literal inválido.
- `GENERATED ALWAYS AS IDENTITY` está **solo en `note`**: el SQL produce `int PRIMARY KEY` sin generador → todo `INSERT` sin ID falla (~35 tablas).

```diff
- created_at timestamptz [not null, default: 'now()']
+ created_at timestamptz [not null, default: `now()`]
- issue_date date [not null, default: 'CURRENT_DATE']
+ issue_date date [not null, default: `CURRENT_DATE`]
- location_id int [pk, note: 'GENERATED ALWAYS AS IDENTITY']
+ location_id int [pk, increment, note: 'GENERATED ALWAYS AS IDENTITY']
```

### C2. `uuid_generate_v7()` no existe con lo declarado
El header exige "PostgreSQL 13+ con pgcrypto", pero `uuid_generate_v7()` **no está en pgcrypto**. Requiere PG 18 (`uuidv7()`), la extensión `pg_uuidv7`, o generarlo en .NET (`Guid.CreateVersion7()`, .NET 9+). Además el código actual usa `Guid.NewGuid()` (v4) → ver D1.

### C3. Unicidades globales que rompen el multi-tenancy
Un mismo paciente/médico en dos clínicas colisiona, y el error de unicidad **revela la existencia del registro en otro tenant** (oráculo de enumeración que RLS no tapa).

| Tabla | Actual | Propuesto |
|---|---|---|
| `patients` | `(document_type_id, document_number)` | `(tenant_id, document_type_id, document_number) WHERE deleted_at IS NULL` |
| `practitioners` | `(document_type_id, document_number)`, `medical_license unique` | ídem con `tenant_id`, o practitioner global + `practitioner_roles` por tenant |
| `appointments` | `appointment_number unique` | `(tenant_id, appointment_number)` |
| `invoices` / `payments` | `invoice_number` / `payment_number unique` | `(tenant_id, …)` |
| `price_lists` | `code unique` | `(tenant_id, code)` |

### C4. Sin integridad referencial entre tenants
`appointments.tenant_id = A` puede apuntar a `patient_id` del tenant B: las FKs son simples. Patrón recomendado: composite FKs.
```sql
ALTER TABLE patient.patients ADD UNIQUE (tenant_id, patient_id);
ALTER TABLE appointment.appointments
  ADD FOREIGN KEY (tenant_id, patient_id) REFERENCES patient.patients (tenant_id, patient_id);
```
Además, `locations` tiene `organization_id` **y** `tenant_id` apuntando a la misma tabla: hay que quitar uno o agregar `CHECK (organization_id = tenant_id)`.

### C5. Tablas con PHI sin `tenant_id` (no pueden tener RLS)
- `system.audit_logs`: `old_values`/`new_values` contienen PHI de todos los tenants.
- `archive.appointment_archives`: `original_data jsonb` con la cita completa.
- `system.users`: sin membresía por tenant (no existe `user_tenant_memberships` ni roles).

### C6. La prevención de double-booking no existe en la base de datos
Los índices `(practitioner_id, slot_start_at) unique` **no evitan solapamientos** (09:00–09:30 vs 09:15–09:45). El `EXCLUDE` está solo en `note`. Hace falta:
```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;  -- no declarada en el header
ALTER TABLE scheduling.slots ADD CONSTRAINT ex_slots_practitioner
  EXCLUDE USING gist (practitioner_id WITH =, tstzrange(slot_start_at, slot_end_at) WITH &&)
  WHERE (reservation_status NOT IN ('CANCELLED','EXPIRED'));
```
Agregar también `CHECK (slot_end_at > slot_start_at)`.

### C7. Contradicción de capacidad en los slots
`slots.max_capacity` admite >1, pero `appointments.slot_id` es `unique` → un slot nunca puede tener más de una cita. Y el `EXCLUDE` por `room_id` contradice `rooms.capacity > 1`. Hay que elegir: o 1 slot = 1 cita (y eliminar `max_capacity`), o slots grupales con conteo protegido por `SELECT … FOR UPDATE`/`version`.

### C8. Secretos y PHI en texto plano
- `telehealth_sessions.host_token` / `guest_token`: guardar el hash o no persistirlos.
- `appointments.cancellation_notes` (texto libre): suele contener PHI. Moverlo a `appointment_clinical_details`.
- `patients.document_number`: si se cifra, el índice único necesita un *blind index* (`document_number_hash`).

---

## 🟠 Altos

1. **Laboratorios no modelables.** `practitioner_id NOT NULL` en `schedules`, `slots` y `appointments` impide agendar un examen sin médico. `pricing.services` no tiene `service_type` (CONSULTATION/LAB_TEST/IMAGING), `attributes jsonb` ni instrucciones de preparación (ayuno). Falta el concepto FHIR `ServiceRequest` (orden de examen). **Esto contradice la directiva 1 del arquitecto.**
2. **Alineación FHIR incompleta.** `practitioner_locations` + `practitioner_specialties` deberían unificarse en `PractitionerRole`, y `location_specialties` corresponde a `HealthcareService`. `Schedule.actor` debería ser polimórfico (practitioner | room | equipment).
3. **Catálogos globales vs. tenant.** `pricing.services.code` es único global: una clínica no puede crear sus propios exámenes. Hay que decidir si el catálogo es global, por tenant o híbrido (`tenant_id NULL` = global).
4. **Faltan RBAC y membresías:** `user_type` es un único valor y no hay `roles`, `permissions` ni `user_tenant_roles` (recepcionista de la clínica A ≠ de la B).
5. **"Event sourcing" declarado, pero no existe.** Lo que hay es historial/auditoría. Para .NET falta `outbox_messages` (transactional outbox) para notificaciones y recordatorios.
6. **Concurrencia parcial:** `version` existe solo en `slots` y `appointments`. Faltan en `invoices`, `payments`, `patients` y `practitioner_schedules`. Con Npgsql es preferible `xmin` (`IsRowVersion()`) a un `int` manual.
7. **Facturación incompatible con el SRI (Ecuador, TZ `America/Guayaquil`):** faltan `access_key` (49 dígitos), `establishment_code`, `emission_point`, `authorization_number/date` y la tasa de IVA por línea (servicios de salud 0% vs 15%). Además `invoices.appointment_id NOT NULL` impide facturar paquetes o varias citas. **`invoices.status_code` no tiene FK** (no existe `invoice_statuses`).
8. **Consentimientos LOPDP:** `patient_consents.appointment_id` **no tiene `ref`**. Faltan `revoked_at`, `consent_version` y `channel`, exigidos por la LOPDP (derecho de revocación).
9. **Particionamiento:** `audit_logs` y `appointment_histories` deberían nacer como `PARTITION BY RANGE (operated_at/changed_at)`. Ojo: la PK debe incluir la clave de partición → `(audit_log_id, operated_at)`. Con eso, `archive.appointment_archives` sobra.

---

## 🟡 Medios

1. **Índices de FK y de RLS:** 137 FKs contra 28 índices. Lo crítico: `tenant_id` como columna líder en los índices de cada tabla tenant, e índice de disponibilidad:
   `slots (tenant_id, location_specialty_id, slot_start_at) WHERE reservation_status = 'FREE'`.
2. **Índices inútiles:** `idx_users_active` e `idx_practitioner_active` sobre booleanos. Reemplazarlos por índices parciales.
3. **`users.email`:** `unique` en la columna + índice único "parcial" duplicado; el constraint total anula el soft delete. Usar `UNIQUE (lower(email)) WHERE deleted_at IS NULL` o `citext`.
4. **Enums sin restricción** (varchar libre): `reservation_status`, `user_type`, `gender` (FHIR: male/female/other/unknown), `room_type`, `relationship_type`, `consent_type`, `booking_channel`, `waitlists.status`, `telehealth.status`, `audit_logs.operation`. Necesitan `CHECK` o lookup.
5. **`appointment_statuses.allowed_transitions varchar 'JSON array'`:** mejor tabla `appointment_status_transitions(from_code, to_code)` con FKs, o al menos `jsonb`.
6. **`slots.reservation_status` default `'HELD'`:** un slot recién generado debería ser `FREE` (FHIR: free/busy/busy-unavailable/busy-tentative).
7. **Redundancias propensas a drift:** `appointments.start_at/end_at/appointment_date` duplican el slot (y `appointment_date` es ambigua respecto a la zona horaria). `location_id + specialty_id` duplican `slot.location_specialty_id`. `patient_insurances.payer_id` es derivable de `plan_id`.
8. **Reglas faltantes:**
   - `CHECK end_time > start_time` en schedules.
   - `EXCLUDE` para horarios solapados y para `service_prices` con vigencias solapadas.
   - `CHECK primary_patient_id <> dependent_patient_id`.
   - Montos `>= 0` y `total = subtotal + tax - discount`.
   - `CHECK (password_hash IS NOT NULL OR external_provider_id IS NOT NULL)`.
9. **Unicidad de "primario":** falta índice único parcial `WHERE is_primary` en addresses, contacts, insurances, `practitioner_specialties` y `is_head_office`.
10. **Auditoría y soft delete inconsistentes:** `created_by/updated_by` faltan en `organizations`, `locations`, `rooms`, `schedules`, `slots`, `price_lists` y `patient_insurances`. `deleted_by` falta en `patients` y `practitioners`. `appointments` no tiene `deleted_at`. Se mezclan `is_active` y `deleted_at` sin una semántica definida.

---

## ⚪ Bajos
- **IDs enumerables:** `slot_id`, `location_id` y `room_id` serían IDs secuenciales expuestos en la API (riesgo IDOR). Usar UUIDv7 o un `public_id`.
- **Lookups en secciones equivocadas:** `document_types` está bajo "PRACTITIONER"; `contact_types`, `specialties` y `services` bajo "GEOGRAPHIC".
- **`schedule_exceptions`:** solo admite días completos o un override. Usar `tstzrange` para bloqueos parciales.
- **Faltan FKs de trazabilidad:** `slots` no tiene `schedule_id` (necesario para regenerar slots), y `waitlists` no tiene `location_id`/`service_id`.
- **Geolocalización:** para búsquedas por cercanía (estilo Zocdoc), considerar PostGIS `geography(Point)` en lugar de `decimal` lat/long.
- **Tabla de duraciones:** `default_duration_minutes` es nullable, pero las agendas lo necesitan.

---

## 🔁 Drift DBML ↔ Código (Organization, migración `Initial`)

| Campo | DBML | Migración EF | Estado |
|---|---|---|---|
| `organization_id` | UUIDv7 | `Guid.NewGuid()` (v4) | ❌ |
| `trade_name`, `tax_id`, `website` | nullable | `NOT NULL` | ❌ |
| `document_type_id` | nullable | `NOT NULL` | ❌ |
| `logo_url`, `is_active`, `created_at`, `updated_at`, `deleted_at` | existen | ausentes | ❌ |
| FKs a `countries` / `document_types` / `organization_types` | sí | no | ❌ |
| `tax_id` unique | total | total | ⚠️ ambos deberían ser parciales |

Extra: el namespace de la migración es `Nexus.Infraestructure.Migrations` (typo) y el proyecto es `Nexus.Infrastructure`.

---

## Plan de remediación sugerido
1. **Sprint 0 (bloqueante):** C1, C2, C3, C4, C6, y alinear `Organization` (drift).
2. **Sprint 1:** C5, C7, C8, RBAC/membresías, outbox, `version`/`xmin`.
3. **Sprint 2:** modelo de laboratorio (`ServiceRequest`, actores polimórficos), `PractitionerRole`, SRI, particionamiento.
4. **Continuo:** índices, CHECKs y limpieza de nomenclatura.
