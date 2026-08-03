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
