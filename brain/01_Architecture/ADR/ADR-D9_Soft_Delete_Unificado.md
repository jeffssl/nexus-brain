---
title: "ADR-D9: Soft delete sin is_active duplicado"
fecha: 2026-10-11
tipo: adr
estado: aceptado
tags: [adr, arquitectura, base-de-datos, data-first]
---

# ADR-D9: Soft delete sin is_active duplicado

> [!success] Decisión
> Datos clínicos usan `deleted_at/by`; no se mezcla con `is_active` para la misma semántica.

## Justificación técnica
Evita estados contradictorios y ambigüedad de unicidad parcial.

## Notas
Requisito legal de retención de registros médicos.

Parte de [[Roadmap_Data_First]] (Fase 0). Índice: [[MOC_Arquitectura_Base_de_Datos]].
