---
title: Roadmap del Proyecto
tags:
  - product
  - roadmap
  - nexus
status: in-progress
---

# Roadmap del Proyecto

## Fase 1: Arquitectura y Backend Core (MVP)
- [x] Definición del Modelo DBML (Completado)
- [x] Definición de la Arquitectura Clean Architecture (Completado)
- [x] Implementación de Entidades de Dominio FHIR
- [x] Casos de uso de autenticación y creación de clínicas
- [x] Migraciones iniciales de Base de Datos
- [x] Auditoría Geográfica y Catálogos Semilla (Country, DocumentType, SNOMED)
- [x] Refactorización DDD: Eliminación del monolito Catalog y distribución de dominios

## Fase 2: Agendamiento y APIs
- [x] Generación de Slots disponibles
- [ ] Prevención de Double-booking con Optimistic Locking
- [x] Exposición de APIs REST para el Frontend

## Fase 3: Integración Frontend
- [ ] Configuración del proyecto Web
- [ ] Vistas de búsqueda y reserva
- [ ] Portal para médicos

## Fase 4: Autenticación Externa y Escalado
- [ ] Integración de OAuth (Google, Microsoft, Apple)
- [ ] Optimización de lectura (Vistas materializadas / Redis)

## Fase 3: Re-cimentación Data-First (2026-10-04)
Detalle en [[Roadmap_Data_First]] (auditoría: [[Auditoria_Nexus_DBML_2026-10-04]]).
- [x] Auditoría de `Nexus.dbml`
- [x] Decisión: SQL versionado como fuente de verdad
- [x] F0 Decisiones y fundamentos (ADRs, `infra/database/`, reglas actualizadas)
- [ ] F1 Plataforma BD (roles, RLS, DbUp, Testcontainers)
- [ ] F2 Tenancy e identidad
- [ ] F3 Catálogos
- [ ] F4 Profesionales y pacientes
- [ ] F5 Agendamiento (EXCLUDE + concurrencia)
- [ ] F6 Laboratorio · F7 Facturación · F8 Auditoría/Outbox · F9 Hardening
