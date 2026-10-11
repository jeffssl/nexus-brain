---
title: "ADR-D2: UUIDv7 como clave primaria expuesta"
fecha: 2026-10-11
tipo: adr
estado: aceptado
tags: [adr, arquitectura, base-de-datos, data-first]
---

# ADR-D2: UUIDv7 como clave primaria expuesta

> [!success] Decisión
> UUIDv7 en tablas expuestas por API; `bigint GENERATED ALWAYS AS IDENTITY` solo en tablas hijas internas.

## Justificación técnica
UUIDv7 es ordenable por tiempo (buen locality de índice) y evita IDs enumerables (IDOR).

## Notas
Resuelve el hallazgo C2 y los IDs enumerables de [[Auditoria_Nexus_DBML_2026-10-04]].

Parte de [[Roadmap_Data_First]] (Fase 0). Índice: [[MOC_Arquitectura_Base_de_Datos]].
