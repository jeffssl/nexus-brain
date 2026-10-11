---
title: "Infraestructura y API: Configuración Inicial de EF Core"
tipo: log
estado: activo
fecha: 2026-09-14
tags:
  - backend
  - infrastructure
  - efcore
  - api
  - postgresql
---

# 🧠 Diario de Sesión: 2026-09-14

## Resumen Ejecutivo
Se implementó manualmente la configuración base de Entity Framework Core y el enrutamiento API para la entidad `Organization` bajo el enfoque de **Clean Architecture**, reforzando el aprendizaje a través de un proceso artesanal (TDD manual).

> [!success] Hito Alcanzado
> El sistema ahora cuenta con el puente completo desde la API HTTP (Postman/`.http`) hasta la persistencia en PostgreSQL, procesado a través de CQRS (MediatR).

## Acciones Realizadas

### 1. Configuración de `Nexus.Infrastructure`
- Se instalaron paquetes de EF Core (`Npgsql.EntityFrameworkCore.PostgreSQL`, `Microsoft.EntityFrameworkCore.Design`, `EFCore.NamingConventions`).
- El desarrollador creó manualmente `NexusDbContext` implementando la interfaz `INexusDbContext` provista por la capa `Application`.
- Se configuró el `OrganizationConfiguration` para definir el mapeo de tabla (sin tocar atributos de dominio).
- Se centralizó el registro de dependencias en `DependencyInjection.cs` con un método estático de extensión, aislando el registro del DbContext en Infraestructura usando el _snake_case naming convention_.

### 2. Configuración de `Nexus.Api`
- Se agregó el Connection String a `appsettings.Development.json` mapeado al contenedor de PostgreSQL local.
- Se reescribió `Program.cs` para registrar las inyecciones de `AddApplication()` y `AddInfrastructure()`, y soporte para controladores.
- Se construyó `OrganizationsController` en `Controllers/v1`, exponiendo un endpoint `POST /api/v1/organizations` delegado a MediatR.

### 3. Validaciones de ASP.NET Core
- Se exploró el comportamiento del atributo `[ApiController]` y su devolución automática de respuestas **RFC 7807 (Problem Details)** ante errores de validación de sintaxis/forma, protegiendo a MediatR de peticiones malformadas.
- Se identificó la necesidad futura de implementar `IExceptionHandler` (Global Exception Handler) para atrapar las excepciones de dominio (errores de fondo).

## Próximos Pasos (Next Steps)
1. Escribir pruebas unitarias o de integración automatizadas para este flujo (Test Automation).
2. Construir e implementar el **Global Exception Handler** para manejar excepciones de dominio mediante respuestas uniformes con Problem Details.
3. Avanzar con la siguiente entidad (ej. `Location` o `Practitioner`) replicando esta misma estructura.
