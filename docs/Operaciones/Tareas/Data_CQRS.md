---
title: Data CQRS
tipo: tarea
estado: todo
fecha: 2026-07-18
prioridad: 2
tags:
  - architecture
  - performance
  - search
---

# Data CQRS (Command Query Responsibility Segregation)

Crear base de datos paralela de solo lectura o índices en **Elasticsearch/Typesense** para búsqueda hiper-rápida de doctores por disponibilidad. 

**Objetivo:** Evitar saturar la DB transaccional con querys de pacientes buscando horas.
