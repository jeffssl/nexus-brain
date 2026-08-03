---
fecha: 2026-07-31
tags:
  - log
  - arquitectura
  - ddd
  - multi-tenancy
tipo: log
---

# Diario del Agente: 31 de Julio de 2026

## 1. Refactorización [[DDD]]: Descentralización de Catálogos
> [!success] Eliminación del Monolito Catalog
> En conjunto con el usuario y el arquitecto de base de datos, se tomó la decisión crítica de eliminar por completo el esquema `catalog`. Mantener todos los diccionarios en un solo esquema generaba acoplamiento ([[Spaghetti Architecture]]) en un diseño que aspira a ser [[Microservices]] / [[Modular-Monolith]] Ready.

- **Migración de Entidades**: 11 entidades fueron físicamente movidas a sus *Bounded Contexts* nativos:
  - `AppointmentStatus`, `CancellationReason` -> Dominio `Appointment`
  - `Specialty`, `SpecialtyCategory` -> Dominio `Practitioner`
  - `PaymentMethod`, `PaymentStatus`, `RefundStatus` -> Dominio `Billing`
  - `Service`, `Currency` -> Dominio `Pricing`
  - `DocumentType` -> Dominio `Geographic`
  - `ContactType` -> Dominio `System`
- **[[Entity Framework Core]]**: Se actualizaron los Namespaces y la instrucción `builder.ToTable("x", "nuevo_esquema")`. Se generó la migración `20260801031000_RemoveCatalogSchema` que ejecutó transparentemente las sentencias `ALTER TABLE x SET SCHEMA y` preservando la integridad referencial y los datos (Zero Data Loss).
- **Documentación**: El Cerebro y los diccionarios ([[Nexus.dbml]], [[Diccionario_de_Datos]]) fueron sincronizados para eliminar referencias al esquema catalog. Se actualizó el [[Changelog_Base_Datos]].

## 2. Decisiones Arquitectónicas: Identidad vs Personas ([[Multi-Tenancy]])
Se documentó y explicó al usuario el patrón para el manejo de usuarios que interactúan en múltiples facetas (Ej: Médico privado en su consultorio, Médico empleado en un Hospital, y Paciente en una clínica externa).

- **Patrón "Identidad Centralizada / Personas Descentralizadas"**:
  - `system.users`: Contiene la autenticación global ([[Single Sign-On]]). El usuario es **uno solo**.
  - `practitioner.practitioners` / `patient.patients`: Son las "Personas" o Perfiles Descentralizados. Un mismo `user_id` puede tener múltiples perfiles de Practitioner (cada uno en un Tenant/Organización distinto para aislar [[RLS]] y precios/horarios independientes) y al mismo tiempo puede tener un perfil de Patient.
- **Ventaja**: Garantiza total privacidad de datos cruzados ([[HIPAA]] compliance), previene mezcla accidental de contextos y proporciona una experiencia de inicio de sesión único y federado.
