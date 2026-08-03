---
fecha: 2026-07-25
tags: [log, architecture, efcore, rls, multi-tenancy]
tipo: log
---

# Diario del Agente - 2026-07-25

> [!info] 
> Resumen de la sesión: Refactorización masiva de Multi-Tenancy (RLS) en la Base de Datos, solución de errores de concurrencia y validación de las Fases 1 y 2 de UAT.

## 1. Refactorización Multi-Tenancy (RLS)
Se completó la adición del campo `TenantId` (UUID) en **todas** las entidades transaccionales (hijas) de la base de datos para habilitar políticas estrictas de Row-Level Security (RLS).
- **Decisión Arquitectónica**: Adoptamos un modelo de *Pool-based Multi-tenancy* donde el aislamiento lógico es gestionado por RLS en PostgreSQL, usando `TenantId`.
- **Implementación**: Modificamos el modelo de EF Core e insertamos explícitamente `TenantId` en entidades del módulo de Sede (`Room`, `Equipment`), Médicos (`PractitionerLocation`, `PractitionerSpecialty`, etc.), y Agendamiento (`PractitionerSchedule`, `Slot`, etc.). 
- **Catálogos**: Se acordó que entidades como `Service`, `Specialty`, `OrganizationType` no llevarán `TenantId` al ser catálogos compartidos globalmente.

## 2. Solución de Fallos en UAT Fase 1 y 2
- Durante las pruebas (UAT Fase 1) vía Postman, la creación de `Room` fallaba silenciosamente en PostgreSQL al omitirse el `tenant_id`.
- Se detectó que el binario de la API (`Nexus.Api`) en ejecución (`task-298`) no contemplaba la nueva estructura.
- **Acción**: Reiniciamos el proceso de la API local (vía `dotnet run`) para compilar los cambios.
- Modificamos los comandos de CQRS (`CreateRoomCommand`, `CreatePractitionerCommand`, `CreatePractitionerScheduleCommand`, `GenerateSlotsCommand`) para que heredaran explícitamente `TenantId` de sus entidades raíz/contexto.

## 3. Documentación Obligatoria (Bilingüe)
- **Decisión**: Se determinó como regla estricta que todo nuevo código de C# deberá tener su respectiva documentación XML (`///`) bilingüe (Español e Inglés) para favorecer la claridad y mantenimiento a futuro. Esta regla fue aplicada a los endpoints de la Fase 2 del UAT.

## Comandos Ejecutados
- `dotnet ef migrations add StrictMultiTenancy`
- `dotnet ef database update`
- `dotnet run --project Brain/Agendamiento-Clinicas-Laboratorios/01_Departamentos/Desarrollo/Backend/src/Nexus.Api`
- Varios comandos de `curl` para validación y generación de Tokens JWT (UAT manual).

> [!success]
> Las validaciones end-to-end (UAT) de la Fase 1 (Sedes y Consultorios) y Fase 2 (Médicos y Horarios) han concluido con éxito insertando `TenantId` sin vulnerar las reglas NOT NULL.
