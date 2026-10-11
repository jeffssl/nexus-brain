---
title: "ADR-D1: PostgreSQL 18 como motor base"
fecha: 2026-10-11
tipo: adr
estado: aceptado
tags: [adr, arquitectura, base-de-datos, data-first]
---

# ADR-D1: PostgreSQL 18 como motor base

> [!success] Decisión
> Se usa PostgreSQL 18.

## Justificación técnica
`uuidv7()` es nativo; evita extensiones o funciones propias para UUIDv7.

## Notas
Si el despliegue objetivo no soporta PG18, crear función `uuidv7()` en V0004 (ver [[ADR-D2_UUIDv7_como_PK]]).

Parte de [[Roadmap_Data_First]] (Fase 0). Índice: [[MOC_Arquitectura_Base_de_Datos]].
