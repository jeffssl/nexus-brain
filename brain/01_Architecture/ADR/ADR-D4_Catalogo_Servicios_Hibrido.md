---
title: "ADR-D4: Catálogo de servicios híbrido"
fecha: 2026-10-11
tipo: adr
estado: aceptado
tags: [adr, arquitectura, base-de-datos, data-first]
---

# ADR-D4: Catálogo de servicios híbrido

> [!success] Decisión
> `services.tenant_id` NULL = global; con valor = propio de la clínica.

## Justificación técnica
Permite catálogo estándar y que cada clínica cree sus servicios/exámenes.

## Notas
Aplica en F3. Requiere RLS que permita leer filas globales.

Parte de [[Roadmap_Data_First]] (Fase 0). Índice: [[MOC_Arquitectura_Base_de_Datos]].
