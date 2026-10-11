---
title: Changelog de Base de Datos
tipo: log
estado: activo
fecha: 2026-07-24
tags:
  - database
  - changelog
  - migraciones
---

# 🗃️ Changelog y Control de Tablas

Este documento mantiene un registro histórico estricto de cómo evoluciona la base de datos `Nexus`. Cada vez que se crea o modifica una tabla mediante migraciones de EF Core, debe quedar registrado aquí.

## [2026-08-06] - Strict Foreign Key Enforcement y Semilla Geográfica

### Añadido / Modificado
- Se analizaron y sincronizaron **todas** las relaciones (Foreign Keys) del archivo `Nexus.dbml` hacia las 38 configuraciones de `IEntityTypeConfiguration` de Entity Framework Core.
- Se implementó la restricción estricta de llaves foráneas (`HasOne().WithMany().HasForeignKey()`) en toda la base de datos para evitar inserciones huérfanas de catálogos y referencias.
- Migración `20260807024322_SyncFKs` generada (más de 1,100 líneas de definiciones de llaves foráneas) y aplicada exitosamente (requirió Drop/Recreate de la base de datos local para limpiar inconsistencias).
- Se solucionaron errores en comandos de CQRS (`CreateOrganizationCommand`, `CreateLocationCommand`) detectados gracias a la validación estricta de la DB.
- Migración `20260807030056_SeedGeographicGuayaquil` generada para inyectar datos de prueba (`Guayas`, `Guayaquil`) y permitir pruebas E2E manuales en Postman.

## [2026-07-31] - Refactorización DDD: Descentralización de Catálogos

### Modificado / Movido
- Se eliminó el monolito del esquema `catalog` (Spaghetti Architecture) moviendo físicamente sus 11 entidades a sus respectivos Bounded Contexts para cumplir con un diseño Microservices/Modular-Monolith.
- **Cambios de Esquema (ALTER TABLE SET SCHEMA):**
  - `AppointmentStatus`, `CancellationReason` pasaron al esquema `appointment`.
  - `Specialty`, `SpecialtyCategory` pasaron al esquema `practitioner`.
  - `PaymentMethod`, `PaymentStatus`, `RefundStatus` pasaron al esquema `billing`.
  - `Service`, `Currency` pasaron al esquema `pricing`.
  - `DocumentType` pasó al esquema `geographic`.
  - `ContactType` pasó al esquema `system`.
- Migración `20260801031000_RemoveCatalogSchema` generada y aplicada.

## [2026-07-25] - Sincronización Total (Zero-Gap)

### Añadido
- Generación de **54 tablas completas** según el diseño oficial en DBML.
- Esquemas creados: `tenant`, `catalog`, `geographic`, `insurance`, `practitioner`, `patient`, `scheduling`, `appointment`, `pricing`, `system`, `billing`, `communication`, `clinical`.
- Migración `InitialCreate` generada y aplicada.

## [2026-07-26] - Rediseño de Especialidades (SNOMED CT) y Categorías

### Añadido / Modificado
- Se creó la tabla `catalog.specialty_categories` para agrupar las especialidades en las 8 divisiones principales (Medicina, Cirugía, Diagnóstico, Salud Mental, Rehabilitación, Odontología, Terapias, Otras).
- La tabla `catalog.specialties` fue rediseñada para usar el estándar mundial SNOMED CT (`snomed_code` VARCHAR(20) UNIQUE) en lugar de códigos internos.
- Se actualizaron los campos a formato bilingüe: `name_es` y `name_en`.
- Se agregó la Foreign Key `category_id` en `specialties` apuntando a `specialty_categories`.
- Migración `RedesignSpecialtiesAndCategories` generada y aplicada.
- **Seed Data**: Se insertaron 8 categorías y 87 especialidades oficiales extraídas de SNOMED CT.

## [2026-07-26] - Refactorización de Tipos de Documento y Catálogos

### Modificado
- La tabla `document_types` fue trasladada estructuralmente del esquema `practitioner` al esquema global `catalog`. Esto corrige una deuda técnica donde entidades como Pacientes requerían acceso a tipos de documentos transversales.
- **`organizations`**: Se añadió la columna `document_type_id` (`integer`) para referenciar adecuadamente el tipo de documento del `TaxId` (ej. RUC, NIT).
- Migración `AuditAndRefactorCatalogs` generada y aplicada.
- **Seed Data**: Se inyectaron registros iniciales para los catálogos `catalog.document_types` (Cédula, Pasaporte, RUC, NIT, SSN) y `catalog.specialties` (10 especialidades médicas).

## [2026-07-26] - Strict Multi-Tenancy

### Modificado
- Se añadió la columna `tenant_id` y su respectivo índice en todas las tablas hijas para asegurar la correcta segregación de datos por inquilino mediante RLS.
- Tablas afectadas: `patient_contacts`, `patient_consents`, `patient_addresses`, `location_specialties`, `location_contacts`, `location_addresses`, `invoice_line_items`, `equipments`, `appointment_histories`, `appointment_clinical_details`, `waitlists`, `telehealth_sessions`, `service_prices`, `schedule_exceptions`, `rooms`, `refunds`, `price_lists`, `practitioner_specialties`, `practitioner_locations`, `practitioner_contacts`, `patient_relations`, `patient_insurances`.
- Migración `StrictMultiTenancy` generada y aplicada exitosamente.

## [2026-07-24] - Inicio de Tracking

### ✨ Tablas Creadas
Se materializaron los esquemas lógicos en tablas físicas en PostgreSQL.

#### Esquema: `organization`
- **`organizations`**: Creada con `organization_id` (UUID), `legal_name`, `tax_id` (índice único), y campos de auditoría (`created_at`, `updated_at`, `deleted_at`).
- **`locations`**: Creada con `location_id`, `organization_id` (FK a `organizations` ON DELETE CASCADE), `tenant_id` y `code` (índice único compuesto `organization_id` + `code` con filtro `deleted_at IS NULL`).

#### Esquema: `practitioner`
- **`practitioners`**: Creada con `practitioner_id`, `tenant_id`, `document_type_id`, `document_number`, `medical_license`. 
  - Índices únicos: `document_type_id` + `document_number` (filtro `deleted_at IS NULL`), y `medical_license`.
  - Índice de búsqueda: `last_name` + `first_name`.

### 🛡️ Políticas Aplicadas
- **Soft Deletes**: Todas las tablas principales incluyen `deleted_at` (TIMESTAMPTZ).
- **Convenciones**: Todas las tablas y columnas usan `snake_case`.


## 2026-10-04 — Auditoría y cambio de estrategia
- Sin cambios de esquema físico. Auditoría de `Nexus.dbml`: [[Auditoria_Nexus_DBML_2026-10-04]].
- Decisión: Data-First; se descartará la migración EF `Initial` y el DBML pasa a ser generado. Ver [[Roadmap_Data_First]].
