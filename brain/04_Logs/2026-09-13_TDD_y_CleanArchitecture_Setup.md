---
fecha: 2026-09-13
tags: [log, tdd, clean-architecture, cqrs, mediatr, nsubstitute, learning]
tipo: log
---

# 🧠 Diario de Arquitectura: Setup de Clean Architecture y TDD (Fase Learning)

> [!info] Resumen de la Sesión
> El usuario solicitó reconstruir la solución del backend en .NET desde cero (`src/backend`) con un enfoque didáctico. El objetivo fue comprender la filosofía detrás de cada comando y aplicar el ciclo **TDD (Test-Driven Development)** de forma manual, validando capa por capa los fundamentos de Clean Architecture.

## Decisiones y Comandos Ejecutados

1. **Estructuración de Clean Architecture (CLI)**
   - Se utilizaron comandos `dotnet new classlib` y `dotnet new webapi` para crear la separación de responsabilidades: `Domain`, `Application`, `Infrastructure`, `Api` y sus respectivos proyectos de `Tests` (xUnit).
   - Se explicó el uso de banderas CLI (`-n` para Name, `-o` para Output) para el scaffolding correcto.
   - **Regla de Dependencias:** El flujo de referencias apunta hacia adentro (Application -> Domain, Infrastructure -> Application, Api -> Application & Infrastructure).

2. **TDD en la Capa de Dominio (Fase 1)**
   - **Herramienta:** `FluentAssertions` para un lenguaje natural en las aserciones.
   - **RED:** Se escribió `OrganizationTests.cs` simulando una creación inválida de clínica sin `LegalName`, forzando errores del compilador (`CS0246`).
   - **GREEN:** Se creó la entidad `Organization` asegurando la inyección y validación del `LegalName` en el constructor.
   - **REFACTOR:** Se añadió documentación XML bilingüe según las reglas del proyecto (`AGENTS.md`).

3. **TDD en la Capa de Aplicación (Fase 2) y CQRS**
   - **Herramientas:** `MediatR` para manejo de comandos y `NSubstitute` para mocks.
   - Se diseñó la abstracción de persistencia con `INexusDbContext` (DbSet).
   - Se implementó `CreateOrganizationCommand` y su Handler.
   - **Mocking:** Se simuló la base de datos inyectando `Substitute.For<INexusDbContext>()` al handler para evitar acoplamiento y lentitud en los tests.

4. **Conceptos Fundamentales Explicados**
   - **Asincronismo y EF Core:** Se desglosó la firma `Task<int> SaveChangesAsync(CancellationToken)`. Se aclaró la diferencia entre tener datos en memoria (Tracked entities) y persistirlos en la base de datos real.
   - **CancellationToken:** Su importancia vital para abortar transacciones de base de datos si la conexión HTTP del cliente se corta, optimizando recursos del servidor.
   - **Interfaces vs Clases:** Se corrigió el error `Non-abstract and non-extern method must declare a body` provocado por definir `INexusDbContext` como `class` en lugar de `interface`.

> [!success] Próximos Pasos
> El usuario completó exitosamente los ciclos TDD básicos y el setup arquitectónico en memoria. El proyecto compila y los tests están en verde. 
> Siguientes retos sugeridos: 
> 1. Implementación de la Infraestructura (PostgreSQL + Entity Framework Core real).
> 2. Exposición del comando a través del Controller en `Nexus.Api`.
