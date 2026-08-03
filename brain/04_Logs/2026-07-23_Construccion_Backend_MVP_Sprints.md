---
title: Diario - 23 de Julio 2026
tipo: log
fecha: 2026-07-23
tags:
  - log
  - backend
  - efcore
  - clean-architecture
  - fhir
  - nexus
---

# 📅 Diario del Agente: 23 de Julio de 2026

Hoy ha sido un día fundamental para el proyecto. Pasamos de la etapa de diseño de datos teórica a la materialización completa de la arquitectura Core en .NET 10. A continuación, el desglose técnico y detallado de todas las decisiones y componentes construidos.

## 1. Re-branding Oficial a "Nexus"
Dejamos atrás el nombre genérico "Agendamiento" para adoptar **Nexus** como nombre oficial del ecosistema. 
- **Documentación Obsidian**: Inyectamos propiedades YAML (Frontmatter) como `title`, `tags`, y `status` en todos los documentos de Producto y Negocio (`Historias_de_Usuario.md`, `Reglas_de_Negocio.md`, `Roadmap.md`). Esto permite que la bóveda de Obsidian funcione como un CMS indexable mediante Dataview u otras integraciones.
- Se revalidó la coherencia con el `Diccionario_de_Datos.md` y el modelo físico.

## 2. Inicialización de Clean Architecture (Backend)
Configuramos la solución .NET (`Nexus.slnx`) respetando de forma estricta los principios de **Clean Architecture**:
- `Nexus.Domain`: Contiene las reglas empresariales puras (entidades y value objects). Cero dependencias externas.
- `Nexus.Application`: Contendrá los Casos de Uso (CQRS con MediatR) y los validadores (FluentValidation).
- `Nexus.Infrastructure`: Encargada de la persistencia de datos (Entity Framework Core) y acceso a servicios externos.
- `Nexus.Api`: La capa de presentación (Minimal APIs / Controllers).
- Se añadieron los proyectos de pruebas unitarias (`Nexus.Domain.Tests` y `Nexus.Application.Tests`) usando xUnit.

## 3. Construcción del Dominio (Estándar FHIR)
Diseñamos el núcleo del negocio asegurando un diseño a prueba de balas:
- **Interfaces Base de Seguridad**:
  - `ITenantIsolated`: Toda entidad que pertenezca a una clínica debe implementar esto. Garantiza que contenga un `TenantId`.
  - `IAuditableEntity`: Exige fechas de `CreatedAt` y `UpdatedAt`.
  - `ISoftDeletable`: Exige una fecha `DeletedAt`. Imprescindible para no borrar historial médico (HIPAA).
- **Anti-Primitive Obsession (Value Objects)**: Creamos la estructura inmutable `readonly record struct TenantId`. En lugar de pasar `Guid` de un lado a otro (con el riesgo de confundirlo con un UserId u otro Guid), obligamos al compilador a trabajar estrictamente con `TenantId`.
- **Entidades de Dominio Centrales**: Construimos `Organization` (Hospital/Clínica), `Location` (Sucursal física) y `Practitioner` (Médico/Personal), modeladas semánticamente basándonos en los recursos de salud de HL7 FHIR.

## 4. Persistencia e Infraestructura (Entity Framework Core)
Esta capa requería cumplir con los estrictos requerimientos de PostgreSQL, el DBML previo y las políticas de seguridad:
- **Naming Conventions**: Integramos `EFCore.NamingConventions` para que C# (`PascalCase`) se traduzca automáticamente a SQL puro (`snake_case`). Un `CreatedAt` en C# es físicamente `created_at` en Postgres.
- **Configuraciones de Mapeo (Fluent API)**:
  - Creamos clases como `OrganizationConfiguration` para aislar el mapeo de cada tabla.
  - Inyectamos **Global Query Filters** (`HasQueryFilter(e => e.DeletedAt == null)`). De esta forma, ninguna consulta LINQ (ya sea de lectura o reportes) arrojará registros que han sido borrados lógicamente.
- **TenantId Converter**: Le instruimos a EF Core cómo serializar nuestro Value Object `TenantId` a un simple `Guid` (tipo `uuid` en Postgres) al guardar, y deserializarlo al consultar, a través de `ValueConverter`.
- **AuditableEntityInterceptor (La Joya de la Seguridad)**:
  - Intercepta el evento de `SavingChanges`.
  - En inserciones, sella la propiedad `CreatedAt`.
  - En actualizaciones, sella la propiedad `UpdatedAt`.
  - **Secuestro de Eliminaciones**: Si el DbContext intenta ejecutar un comando DELETE en la base de datos, el interceptor aborta la eliminación física, cambia el estado a `Modified`, y simplemente estampa la fecha actual en `DeletedAt`. (Cumplimiento de borrado lógico por defecto).

## 5. Pruebas de Humo y Migraciones
Para coronar la arquitectura, unimos todas las piezas:
- Registramos la inyección de dependencias `NexusDbContext` y los interceptores en `Nexus.Infrastructure/DependencyInjection.cs`.
- Conectamos la configuración a `Nexus.Api` en el archivo `Program.cs` mediante `builder.Services.AddInfrastructureServices`.
- Establecimos nuestra cadena de conexión inicial hacia PostgreSQL en el `appsettings.json`.
- Instalamos la herramienta `dotnet-ef` globalmente en el repositorio e instalamos la librería `Microsoft.EntityFrameworkCore.Design` en el proyecto API.
- Ejecutamos con éxito `dotnet ef migrations add InitialCreate`. EF Core analizó todo nuestro dominio, aplicó los convenios snake_case, resolvió los tipos, acató los Soft Deletes y produjo el código C# y SQL perfecto para la primera migración de la base de datos.

> [!success] Hito Fase 1 Alcanzado
> Hemos logrado un andamiaje completo, fuertemente tipado, auditable y seguro (preparado para Multi-Tenancy y HIPAA). La base de datos es ahora capaz de alojar operaciones complejas con total garantía.
> **Próximo Paso**: Capa de Aplicación, MediatR, CQRS y Casos de Uso (Fase 2).

---

## 6. Motor de Aplicación y CQRS (Fase 2 Completada)
Dimos un salto fundamental construyendo la **Capa de Aplicación (Application Layer)** encargada de orquestar absolutamente toda la lógica de negocio, de nuevo, de manera agnóstica a la infraestructura.
- **Implementación de MediatR**: Instalamos y configuramos el patrón Mediator y CQRS. Cada acción en el sistema es ahora un `Command` (o `Query`) con su respectivo `Handler`.
- **Validation Pipeline (Interceptor)**: Creamos un `ValidationBehavior` que se inyecta en el pipeline de MediatR. Este interceptor captura automáticamente los Comandos antes de ejecutarse, evalúa las reglas de `FluentValidation` y, si los datos son inválidos o maliciosos, detiene la ejecución arrojando una `ValidationException` personalizada (preparando el terreno para retornar 400 Bad Requests puros en la API).
- **Casos de Uso Iniciales (Handlers)**:
  - `CreateOrganizationCommand`: Permite dar de alta a una Clínica. Su validador exige un nombre legal, un país (`CountryCode`) y longitud máxima de campos. Su Handler usa el Factory del Dominio y lo persiste usando una interfaz purista (`INexusDbContext`).
  - `CreatePractitionerCommand`: Permite dar de alta a médicos. Validamos que reciba un `TenantId` obligatorio para asegurar que el médico nazca fuertemente amarrado a una organización.
- **Independencia Total**: Configuramos `INexusDbContext` dentro de `Nexus.Application/Common/Interfaces` para que los Casos de Uso interactúen con la persistencia sin saber que por detrás existe Entity Framework Core. El acoplamiento es igual a cero.

> [!success] Hito Fase 2 Alcanzado
> Hemos finalizado la capa de Aplicación con un patrón CQRS robusto, validaciones en línea y un flujo fuertemente tipado. La lógica de negocio está terminada para nuestros dos casos de uso core.
> **Próximo Paso**: Capa API Web (REST) para conectar estos comandos al mundo exterior mediante Minimal APIs (Fase 3).

---

## 7. Capa API y Exposición REST (Fase 3 Completada)
Con la lógica de negocio asegurada por MediatR, construimos la **Capa de Presentación** (`Nexus.Api`) para exponer las capacidades del sistema a futuros frontends o clientes HTTP, asegurando alto rendimiento y manejo robusto de excepciones.

- **Minimal APIs y Route Groups**: En lugar de usar los pesados `Controllers` tradicionales, utilizamos la sintaxis declarativa de Minimal APIs (`app.MapGroup`). Esto reduce la latencia y hace que el enrutamiento de C# sea extremadamente rápido e idiomático.
- **Mapeo Directo (Endpoints)**:
  - Creamos `OrganizationsEndpoints` (`POST /api/v1/organizations`) y `PractitionersEndpoints` (`POST /api/v1/practitioners`). 
  - La API de .NET se encarga de serializar el JSON entrante hacia el `Record` correspondiente (ej. `CreateOrganizationCommand`) e inyectarlo automáticamente como dependencia en los endpoints, que a su vez lo reenvían al bus de MediatR.
- **Global Exception Handler (RFC 7807)**:
  - Para evitar fugas de información y estandarizar las respuestas de error en HTTP 400 y 500, implementamos el nuevo pipeline `IExceptionHandler` de .NET.
  - El archivo `GlobalExceptionHandler.cs` intercepta específicamente las `ValidationException` arrojadas por nuestra capa de Aplicación y las transforma en una respuesta estándar de la industria (`ValidationProblemDetails`), incluyendo la lista exacta de campos que fallaron, simplificando la vida a los desarrolladores Frontend que consuman nuestra API.
- **Limpieza de Host**: Configuramos `Program.cs` para registrar nuestros servicios de infraestructura, aplicación y manejo de errores limpiamente y removimos el "WeatherForecast" por defecto.

> [!success] Hito Fase 3 Alcanzado
> El esqueleto "Backend MVP" está listo al 100%. Los datos pueden entrar desde un JSON por HTTP, validarse de manera estricta, ejecutar la lógica de FHIR y persistirse correctamente en PostgreSQL respetando Multi-Tenancy y Soft Deletes.
> **Próximo Paso**: Aplicar físicamente las migraciones de base de datos (`dotnet ef database update`) y probar el servidor en caliente.
