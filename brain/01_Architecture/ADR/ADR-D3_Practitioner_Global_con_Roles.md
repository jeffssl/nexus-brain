---
title: "ADR-D3: Médico global con PractitionerRole por tenant"
fecha: 2026-10-11
tipo: adr
estado: aceptado
tags: [adr, arquitectura, base-de-datos, data-first]
---

# ADR-D3: Médico global con PractitionerRole por tenant

> [!success] Decisión
> `practitioners` es global; `practitioner_roles` vincula médico, tenant, sede y especialidad (FHIR PractitionerRole).

## Justificación técnica
Un médico atiende en varias clínicas sin duplicar su identidad; evita colisiones de unicidad.

## Notas
Aplica en F4. Relacionado con [[ADR-D8_RLS_con_current_tenant]].

Parte de [[Roadmap_Data_First]] (Fase 0). Índice: [[MOC_Arquitectura_Base_de_Datos]].
