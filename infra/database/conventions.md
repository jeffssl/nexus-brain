# Convenciones de base de datos (F0)

## Nomenclatura
- `snake_case`; tablas en plural; un schema por dominio.
- Migraciones: `V0001__descripcion_corta.sql`. **Inmutables** una vez fusionadas; se corrige con una nueva.
- Seeds idempotentes: `INSERT ... ON CONFLICT DO NOTHING`.

## Reglas por tabla de tenant
- `tenant_id uuid NOT NULL` + `UNIQUE (tenant_id, <pk>)`.
- RLS `ENABLE` + `FORCE` con política sobre `current_tenant_id()`.
- FKs entre tablas de tenant: **compuestas** `(tenant_id, x_id)`.
- Toda FK con índice; `tenant_id` primera columna de los índices.
- Enums como `CHECK` o tabla lookup (nunca `varchar` libre).
- Auditoría: `created_at/by`, `updated_at/by`; `deleted_at/by` en datos clínicos (sin `is_active` duplicado).
- Tipos: `timestamptz`, `numeric`, `jsonb`, `citext`. IDs: UUIDv7 (`uuidv7()`), `bigint identity` solo en hijas internas.
- Defaults con funciones reales (`now()`, `CURRENT_DATE`), nunca literales.

## Roles
`nexus_owner` (migraciones) · `nexus_app` (DML, sin superusuario, no dueño) · `nexus_readonly`.

## Definition of Done por tabla
Ver sección homónima en `brain/01_Architecture/Roadmap_Data_First.md`.
