---
title: "ADR-D5: 1 slot = 1 cita"
fecha: 2026-10-11
tipo: adr
estado: aceptado
tags: [adr, arquitectura, base-de-datos, data-first]
---

# ADR-D5: 1 slot = 1 cita

> [!success] Decisión
> Cada slot admite una sola cita; slots grupales se difieren.

## Justificación técnica
Simplifica la integridad (`EXCLUDE` + unicidad) y elimina la contradicción `max_capacity` vs `slot_id` único.

## Notas
Se eliminará `max_capacity` o se reintroducirá solo con un diseño de conteo protegido.

Parte de [[Roadmap_Data_First]] (Fase 0). Índice: [[MOC_Arquitectura_Base_de_Datos]].
