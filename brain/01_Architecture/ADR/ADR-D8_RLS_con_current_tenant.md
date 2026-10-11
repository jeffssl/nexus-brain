---
title: "ADR-D8: RLS con current_setting y FORCE"
fecha: 2026-10-11
tipo: adr
estado: aceptado
tags: [adr, arquitectura, base-de-datos, data-first]
---

# ADR-D8: RLS con current_setting y FORCE

> [!success] Decisión
> Aislamiento por `current_setting('app.tenant_id')` con `ENABLE` y `FORCE ROW LEVEL SECURITY`; la app usa rol no dueño.

## Justificación técnica
El aislamiento queda garantizado por el motor aunque falle la capa de aplicación.

## Notas
Un rol dueño anularía RLS: por eso `nexus_app` no es dueño (F1).

Parte de [[Roadmap_Data_First]] (Fase 0). Índice: [[MOC_Arquitectura_Base_de_Datos]].
