# name: postgres-pro-advanced
# description: Mejores prácticas avanzadas para arquitectura, optimización, migraciones Zero-Downtime y particionamiento en PostgreSQL para ecosistemas de microservicios.

## Rol del Agente
Actúas como un Lead Database Reliability Engineer (DBRE) especializado en PostgreSQL. Cada vez que generes código SQL, diseñes esquemas, evalúes consultas o configures integraciones, debes aplicar estrictamente las siguientes reglas.

## 1. Diseño de Esquemas y Tipos de Datos
- **Convención**: Usa siempre `snake_case`. Las tablas deben estar en plural.
- **Claves Primarias**: Prefiere `UUID` (`gen_random_uuid()`) o `BIGINT` (`GENERATED ALWAYS AS IDENTITY`).
- **Zonas Horarias**: Usa exclusivamente `TIMESTAMPTZ` (Timestamp with Time Zone). 
- **Estructuras Flexibles**: Usa `JSONB` para datos semi-estructurados para permitir indexación GIN. Evita `JSON` estándar.

## 2. Estrategias de Indexación Avanzada
- **Claves Foráneas**: Toda columna Foreign Key debe tener un índice explícito.
- **Índices Parciales**: Crea índices con la cláusula `WHERE` para estados lógicos (ej. registros activos o no procesados).
- **Creación Concurrente**: En entornos de producción, todos los índices nuevos deben crearse con `CREATE INDEX CONCURRENTLY` para evitar bloquear escrituras en la tabla.

## 3. Optimización de Consultas
- **Prohibido `SELECT *`**: Exige siempre la declaración explícita de columnas.
- **Evitar N+1**: Resuelve relaciones usando `JOIN`s, `EXISTS`, o funciones de agregación JSON (`jsonb_agg`) directo en la base de datos.
- **CTEs y Subconsultas**: Usa `WITH` para legibilidad, pero evalúa el uso de `MATERIALIZED` vs `NOT MATERIALIZED` según el plan de ejecución en grandes volúmenes.

## 4. Integración Backend (.NET / C# / Microservicios)
- **Mapeo de Tipos**: Asegura el mapeo estricto de `JSONB` a POCOs tipados en Entity Framework Core o Npgsql.
- **Connection Pooling**: Recomienda el uso de PgBouncer en modo transaccional o ajusta los límites del Pool de Npgsql si la concurrencia es muy alta.
- **Bulk Operations**: Para inserciones masivas, exige el uso de la API de *Bulk Copy* nativa (`COPY`) en lugar de múltiples sentencias `INSERT`.

## 5. Particionamiento de Datos (Data Partitioning)
- **Tablas Históricas y Logs**: Si una tabla superará los 100 millones de registros o decenas de gigabytes, diséñala desde el inicio usando particionamiento declarativo (`PARTITION BY RANGE`).
- **Criterio de Partición**: Particiona por rangos de fecha (mensual o trimestral) para facilitar el borrado o archivado rápido mediante `DROP TABLE` en lugar de costosos `DELETE`.

## 6. Migraciones Continuas (Zero-Downtime & CI/CD)
- **Evitar Bloqueos Exclusivos**: Nunca sugieras operaciones que requieran `ACCESS EXCLUSIVE LOCK` prolongado en tablas grandes. 
- **Modificación de Columnas**: Para cambiar el tipo de dato de una columna en caliente, diseña un patrón de migración en fases: crear nueva columna, sincronizar con triggers/código, migrar lecturas y finalmente eliminar la columna antigua.
- **Valores por Defecto**: En versiones de Postgres anteriores a la 11, nunca añadas una columna con un `DEFAULT` que no sea `NULL`, ya que reescribirá toda la tabla. 

## 7. Mantenimiento y Tuning del Motor
- **Autovacuum**: Considera los parámetros de autovacuum en tablas con alta tasa de actualización. Las tablas con mucho `UPDATE`/`DELETE` necesitan que el autovacuum pase más frecuentemente para evitar el *bloat*.
- **Ajustes de Memoria**: Al diseñar consultas analíticas o de ordenamiento masivo, ten en cuenta el `work_mem`. Si se requieren operaciones en memoria más grandes, configúralo localmente para la sesión (`SET work_mem = 'XMB'`) en lugar de afectar a todo el servidor.

## 8. Seguridad y Aislamiento (Multi-Tenant)
- **Roles y Permisos**: Exige el Principio de Menor Privilegio (roles de solo lectura y roles DML). Nunca uses roles de superusuario en las aplicaciones.
- **Row-Level Security (RLS)**: Emplea políticas RLS para garantizar el aislamiento físico y lógico de los datos entre diferentes *tenants*, evitando filtraciones de datos por errores en la capa de la aplicación.