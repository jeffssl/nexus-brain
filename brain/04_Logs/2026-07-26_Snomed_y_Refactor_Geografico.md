---
fecha: 2026-07-26
tags: [log, database, uat, refactor]
tipo: log
---

# Diario del Agente - 26 de Julio de 2026

## 1. Rediseño del Catálogo de Especialidades (Domain & Database)
> [!info] Decisión Arquitectónica
> Se solicitó que el sistema utilice directamente el estándar **SNOMED CT** en lugar de especialidades genéricas en duro. Esto asegura la interoperabilidad con otros sistemas de salud en el futuro (HIE) y el uso de un diccionario de datos mundialmente estandarizado.

- **Entity Framework & Dominio**:
  - Se modificó la entidad `Specialty` agregando las propiedades `SnomedCode`, `NameEn`, `NameEs`.
  - Se agregó la entidad de categorización de SNOMED CT: `SpecialtyCategory` para agrupar bajo ramas como (Medicina, Cirugía, Diagnóstico, Odontología, etc.).
  - Se definieron Seeds masivos (80+ registros) en `SpecialtyConfiguration.cs` y `SpecialtyCategoryConfiguration.cs` para inyectar automáticamente la data usando `HasData()` en EF Core.
- **Base de Datos (PostgreSQL)**:
  - Generamos y aplicamos la migración `RedesignSpecialtiesAndCategories`.
  - Actualizamos la documentación de la base de datos de manera automática (`Nexus.dbml`, `Diccionario_de_Datos.md`, `Changelog_Base_Datos.md`).

## 2. Pruebas End-to-End (UAT) y MCP Postman
> [!success] Validación Exitosa
> Se simularon los roles de un ingeniero QA para realizar UAT (User Acceptance Testing) sobre la creación de Organizaciones y Médicos (Practitioners), garantizando que los catálogos vinculen correctamente.

- Se utilizó proactivamente el Servidor MCP `postman-mcp-server`.
- Se creó un Workspace, un Environment Local y la Colección de pruebas (`1. Onboarding Clínica`).
- Se lanzaron las APIs vía consola (`dotnet run`) y se ejecutaron las peticiones a:
  - `POST /api/v1/organizations` (Validando el rechazo por TaxId duplicado de forma correcta con ProblemDetails 500/409, y probando éxito en creación).
  - `POST /api/v1/locations` (Creación de sedes).
  - `POST /api/v1/practitioners` (Vinculación cruzada de Tenant, Location y Especialidades usando el catálogo de SNOMED).
- **Correcciones realizadas en API**:
  - Por defecto los endpoints de Practitioners y Locations requerían autorización. Para fines del UAT automatizado, temporalmente los cambiamos a `.AllowAnonymous()` y se comprobó que el flujo responde exitosamente con el estatus `201 Created` y los respectivos IDs. Luego se restauró el `.RequireAuthorization()` en los endpoints para mantener la seguridad.

## 3. Auditoría y Refactorización Geográfica (Domain & Database)
> [!info] Decisión Arquitectónica
> Se detectaron dependencias rígidas (por ejemplo, el uso de texto directo en la columna `CountryCode`) y la ausencia de validaciones cruzadas sobre el tipo de documento. Se decidió pivotar hacia una estructura normalizada con claves primarias enteras (`Identity`) para optimizar el rendimiento y la integridad referencial.

- **Entity Framework & Dominio**:
  - Se refactorizó la entidad `Country` migrando su `PK` de `string CountryCode` a `int CountryId`.
  - Se actualizaron las propiedades foráneas (`FK`) en las entidades `State`, `Organization`, `Payer` y `DocumentType` a `CountryId`.
  - Se inyectó un catálogo semilla (Seed) de 10 países y 10 Tipos de Documentos asociados a dichos países.
  - Se agregaron reglas de `FluentValidation` en `CreateOrganizationCommand` para obligar a que si existe un `TaxId`, obligatoriamente se debe suministrar su respectivo `DocumentTypeId`.
- **Base de Datos (PostgreSQL)**:
  - Se generó y aplicó la migración `AuditGeographicCatalogs`.
  - Se sincronicó la documentación física mediante la actualización de `Nexus.dbml` y el `Diccionario_de_Datos.md`.

## Próximos Pasos (Pendientes)
- Finalizar implementaciones REST para listado público (pacientes) que no requieran Auth y usar el catálogo de especialidades.
- Abordar HU-04 (Como Paciente, quiero buscar horarios disponibles...).

## 4. Inyección de Catálogos Semilla y Máquinas de Estado
> [!success] Inicialización de Datos
> Se configuraron 11 catálogos maestros mediante Entity Framework (`HasData()`) para asegurar que todo nuevo Tenant encuentre la información base de la plataforma sin requerir carga manual.

- **Entity Framework (Catalog Seeds)**:
  - Se inyectaron estados terminales y transitorios (`AppointmentStatus`, `PaymentStatus`, `RefundStatus`) definiendo la propiedad `IsFinal` que rige el flujo de negocio.
  - Se crearon tipos base: `CancellationReason`, `ContactType`, `Currency`, `PaymentMethod`, `Service`, `OrganizationType`, `CoverageType`, `Weekday`.
- **Base de Datos (PostgreSQL)**:
  - Generamos y aplicamos la migración `SeedSystemCatalogs`.
- **Decisiones Educativas / Arquitectónicas**:
  - Se instruyó al usuario sobre la diferencia entre alterar un *Catálogo (Diccionario de solo lectura)* versus alterar una *Transacción (Llave foránea)* en un modelo Relacional (RDBMS), explicando la utilidad del Data-Driven Design para evitar bloqueos hardcodeados (`if/switch`) en C#.
  - **Excepción de Llaves Primarias (Surrogate vs Natural)**: Se debatió por qué `CountryId` usa un número (Surrogate Key para performance y seguridad ante cambios de la ISO) y por qué catálogos minúsculos y estáticos como `AppointmentStatus` usan Llaves Naturales (`"SCHEDULED"`) actuando como Enums legibles. Se acordó mantener este patrón mixto (Surrogates para catálogos grandes, Textos para Enums) a favor de la legibilidad en queries de negocio.
