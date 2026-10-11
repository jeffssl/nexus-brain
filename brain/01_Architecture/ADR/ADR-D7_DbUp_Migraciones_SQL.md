---
title: "ADR-D7: DbUp con migraciones SQL forward-only"
fecha: 2026-10-11
tipo: adr
estado: aceptado
tags: [adr, arquitectura, base-de-datos, data-first]
---

# ADR-D7: DbUp con migraciones SQL forward-only

> [!success] Decisión
> Migraciones `.sql` en `infra/database/migrations`, ejecutadas por un runner DbUp; EF Core sin migraciones.

## Justificación técnica
Control total de PostgreSQL (RLS, EXCLUDE, particiones, locks).

## Notas
Se descarta la migración EF `Initial`. Ver [[Roadmap_Data_First]].

Parte de [[Roadmap_Data_First]] (Fase 0). Índice: [[MOC_Arquitectura_Base_de_Datos]].
