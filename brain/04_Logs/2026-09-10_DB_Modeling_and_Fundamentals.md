---
fecha: 2026-09-10
tags: [log, dbml, architecture, saas, fhir, rls]
tipo: log
---

# 🧠 Diario de Arquitectura: Modelado DBML y Fundamentos SaaS

> [!info] Resumen de la Sesión
> El usuario solicitó aprender y replicar la arquitectura de base de datos desde cero para entender los fundamentos antes de pasar al código C# (TDD). Se creó un archivo didáctico `Nexus_Learning.dbml` para reescribir y analizar el esquema bloque a bloque de manera interactiva.

## Decisiones Arquitectónicas Revisadas

1. **Multi-Tenancy y Row-Level Security (RLS)**
   - Se analizó la importancia de la columna `tenant_id` vinculada a la tabla `organization.organizations`.
   - **Razón:** Aislar físicamente a nivel de PostgreSQL los datos de diferentes clínicas (ej. Hospital A vs Hospital B) para evitar fugas de datos y cumplir con normativas de salud (HIPAA).

2. **Convenciones de Nomenclatura (Plural vs Singular)**
   - Se estableció la convención de usar **Plural** para las tablas (ej. `users`, `locations`) y **Singular** para las clases/entidades en C#.
   - **Razón:** Evitar colisiones con palabras reservadas de SQL (como `user`) y mantener el estándar moderno de diseño.

3. **Estrategia de IDs (UUIDv7 vs Identity)**
   - Uso de `` `uuid_generate_v7()` `` para entidades públicas (`users`, `organizations`, `practitioners`, `patients`) para garantizar unicidad global y buen rendimiento en índices B-Tree gracias a su componente de ordenamiento por tiempo.
   - Uso de `GENERATED ALWAYS AS IDENTITY` (BigInt/Int) para tablas de soporte interno (`locations`, `slots`) para optimizar el almacenamiento y la velocidad de los JOINs.

4. **Aislamiento de Identidad (Practitioner vs User)**
   - Se clarificó que el registro médico (`practitioner`) es independiente de la cuenta de acceso (`user`). Un médico puede trabajar en dos clínicas distintas, existiendo como dos registros separados de `practitioner` (por RLS), pero vinculados a un solo `user_id` para acceso centralizado (Workspace).
   - **Regla de Negocio:** La seguridad para evitar que un usuario "Pablo" se apropie del perfil del "Dr. Juan" recae estrictamente en la capa de Aplicación (Backend C#), mediante tokens de invitación seguros, no en la base de datos.

5. **Alineación FHIR**
   - El diseño respeta los dominios del estándar HL7 FHIR: `Organization`, `Location`, `Practitioner`, `Patient`, `Slot`, `Appointment`.

6. **Control de Concurrencia y Catálogos**
   - En el módulo de Agendamiento, se discutió el uso de `version int` para **Optimistic Locking**, evitando sobre-reservas (double-booking).
   - Se justificó el uso de Enums (Texto) para estados estructurales inmutables (ej. `reservation_status` en slots) versus Tablas de Catálogo para estados variables con metadatos (ej. `appointment_statuses` en appointments).

> [!success] Próximos Pasos
> Con los cimientos de la base de datos (Etapa 1) comprendidos a la perfección mediante escritura guiada, el usuario está listo para avanzar a la **Etapa 2 (Creación de la Solución .NET Core)** e iniciar la escritura de casos de uso aplicando **TDD**.
