---
title: DB Schema Enforcement y Pruebas Manuales E2E
fecha: 2026-08-06
tipo: log
tags:
  - log
  - database
  - foreign-keys
  - e2e
  - postman
---

# DB Schema Enforcement y Pruebas Manuales E2E

> [!info] Objetivo de la Sesión
> El usuario solicitó realizar pruebas manuales paso a paso (End-to-End) usando Postman para entender el flujo actual del sistema, comenzando por el módulo de Organización (Organization) y Sedes (Locations). Al hacerlo, descubrimos problemas de integridad de datos y nos enfocamos en forzar validaciones estrictas.

## 1. Detección del Problema y Sincronización DBML -> EF Core

Durante las pruebas, nos dimos cuenta de que la base de datos no estaba protegiendo la integridad de sus registros frente a catálogos inexistentes (ej. `location_specialties`). Esto se debió a que, aunque las relaciones existían lógicamente en `Nexus.dbml`, nunca se configuraron mediante `HasOne().WithMany().HasForeignKey()` en los archivos de Entity Framework.

> [!success] Implementación: Strict Foreign Keys
> Se creó y ejecutó un script en Python (`extract_fks.py` e `inject_fks.py`) para analizar el DBML y auto-inyectar la configuración estricta de **todas** las llaves foráneas en los 38 archivos `IEntityTypeConfiguration`.
> - **Migración generada**: `20260807024322_SyncFKs.cs`
> - **Acción drástica pero necesaria**: Se ejecutó `dotnet ef database drop` y `dotnet ef database update` para recrear la base de datos desde cero, garantizando que ninguna tabla huérfana violara las nuevas restricciones.

## 2. Depuración Guiada por Base de Datos

Una vez aplicadas las restricciones de base de datos, el flujo de creación fue bloqueado en repetidas ocasiones por la propia DB (PostgreSQL). ¡Esto es exactamente lo que queríamos (Fail Fast)!

### Error 1: Creación de Organización
- **Síntoma**: Error HTTP 500 con código PostgreSQL `23503` (violates foreign key constraint `fk_organizations_organization_types_organization_type_id`).
- **Diagnóstico**: `CreateOrganizationCommand` no solicitaba el `OrganizationTypeId`, enviando por defecto un `0` a la BD (el cual no existe en el catálogo `organization_types`).
- **Solución**: Se expuso la propiedad `int OrganizationTypeId` en el Record del Request y se modificó `CreateOrganizationCommandHandler` para mapearlo a la entidad de dominio.

### Error 2: Creación de Location (Sede)
- **Síntoma**: Error HTTP 500 con código `23503` al intentar insertar en `location_addresses` (`violates foreign key constraint "fk_location_addresses_cities_city_id"`).
- **Diagnóstico**: El usuario intentó enviar `cityId: 1`, pero el catálogo de ciudades estaba vacío en la base de datos (no existía semilla/seed). Además, descubrimos un bug preventivo en el handler: `TenantId` no se estaba asignando al objeto de `LocationAddress`.
- **Solución**:
  - Se corrigió `CreateLocationCommandHandler.cs` agregando `TenantId = request.TenantId`.
  - Se añadieron datos semilla para Ecuador (`Guayas` y `Guayaquil`) en `StateConfiguration` y `CityConfiguration`.
  - Se aplicó la migración `SeedGeographicGuayaquil`.

## 3. Comandos Ejecutados

Durante la sesión se ejecutaron constantemente los siguientes comandos de infraestructura:
```bash
dotnet ef migrations add SyncFKs --project src/Backend/src/Nexus.Infrastructure/ --startup-project src/Backend/src/Nexus.Api/
dotnet ef database drop --force --project src/Backend/src/Nexus.Infrastructure/ --startup-project src/Backend/src/Nexus.Api/
dotnet ef database update --project src/Backend/src/Nexus.Infrastructure/ --startup-project src/Backend/src/Nexus.Api/
dotnet build
```

## Próximos Pasos (Next Steps)
El sistema ahora obliga a tener una integridad relacional perfecta. El próximo paso de la sesión consistirá en continuar probando la creación de la Location en Postman y avanzar hacia los endpoints de Practitioners y Schedules según los roadmaps de E2E establecidos.
