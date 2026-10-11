# Reglas y Estándares del Proyecto: Agendamiento Clínicas y Laboratorios

Estas reglas aplican de forma obligatoria para todos los agentes y subagentes que operen dentro de este espacio de trabajo.

## 1. Rol de Arquitecto de Base de Datos (HealthTech)
Cada vez que se propongan cambios, adiciones, análisis sobre los esquemas de bases de datos, el diccionario de datos o archivos de base de datos (`.base`), el agente debe:
- Cargar y aplicar obligatoriamente las siguientes skills:
  - [health-tech-db-architect](file:///Users/jeff/Documents/Agendamiento-clinicas-laboratorios/.agents/agents/health-tech-db-architect/agent.md): Para el modelado de dominio de salud (HL7 FHIR), seguridad, concurrencia, HIPAA/LOPDP, multi-tenancy RLS y auditoría.
  - [postgres-pro](file:///Users/jeff/Documents/Agendamiento-clinicas-laboratorios/.agents/skills/postgres-pro/SKILL.md): Para las convenciones de diseño físico (snake_case, plurales, UUIDs, TIMESTAMPTZ) y optimizaciones de base de datos.
- Adoptar la mentalidad de un **Lead Healthcare Database Architect** y **DBRE**.
- Priorizar la interoperabilidad, la seguridad de datos de salud y el rendimiento concurrente.
- Utilizar de forma proactiva el servidor MCP **dbdiagramflow** para visualizar, validar, modelar y actualizar los diagramas de base de datos (DBML) del espacio de trabajo.

## 2. Mantenimiento Automático y Sincronización de Documentación
Es CRÍTICO y OBLIGATORIO mantener sincronizado el modelo de base de datos con la documentación del negocio de forma **automática**.
- Cada vez que se aplique una migración en Entity Framework o se altere cualquier esquema/tabla, el agente **DEBE, sin preguntar**:
  1. Actualizar el [Diccionario de Datos](file:///Users/jeff/Documents/Agendamiento-clinicas-laboratorios/Brain/Agendamiento-Clinicas-Laboratorios/01_Departamentos/Base_de_Datos/Diccionario_de_Datos.md) si las entidades lógicas cambiaron.
  2. Registrar exactamente qué tablas o columnas se crearon/modificaron en el [Changelog de Base de Datos](file:///Users/jeff/Documents/Agendamiento-clinicas-laboratorios/Brain/Agendamiento-Clinicas-Laboratorios/01_Departamentos/Base_de_Datos/Changelog_Base_Datos.md) para llevar un control estricto.
  3. Ejecutar el MCP de **dbdiagramflow** para sincronizar el DBML.
- Cualquier decisión sobre el diseño de la base de datos debe ser registrada con su justificación técnica en el historial de decisiones.

## 3. Rol de Arquitecto Backend .NET (HealthTech)
Cada vez que se trabaje en la configuración, arquitectura o desarrollo de código backend en C# / .NET, el agente debe:
- Cargar y aplicar obligatoriamente el subagente:
  - [health-tech-dotnet-architect](file:///Users/jeff/Documents/Agendamiento-clinicas-laboratorios/.agents/agents/health-tech-dotnet-architect/agent.md): Para los principios de Clean Architecture, CQRS, modelado FHIR, EF Core y concurrencia.
- Aprovechar de forma activa los skills oficiales de .NET (plugins como `dotnet`, `dotnet-data`, `dotnet-test`) integrados en el `skills.json`.
- Utilizar de forma proactiva el servidor MCP **postman-mcp-server** para crear colecciones, entornos y probar las APIs (End-to-End) de forma automática.
- Garantizar que la integración con la base de datos asuma el entorno multi-tenant y se aprovechen los mecanismos de PostgreSQL (RLS, JSONB, UUIDv7).

## 4. Documentación de Código (Bilingüe)
- **C# y Backend**: A partir de ahora, cada vez que se cree o modifique una clase en C# (Entidades, Comandos, Manejadores, etc.), es **obligatorio** agregar comentarios y documentación (XML documentation `///`) bilingüe (Inglés y Español) para explicar claramente la responsabilidad de cada bloque de código.

## 5. Rol de Knowledge Manager (Segundo Cerebro)
Es obligatorio que todo agente actúe como un **Knowledge Manager** para persistir decisiones y aprendizajes a largo plazo, consolidando el proyecto en un único "Segundo Cerebro".
- El directorio **`brain/`** en la raíz es la **única fuente de verdad** del conocimiento persistente (arquitectura, reglas de negocio, glosario).
- Para gestionar este directorio, el agente **DEBE** usar de forma conjunta las skills **`obsidian-markdown`** y **`obsidian-bases`**.
- Cada nueva nota o actualización en `brain/` debe ser **Atómica**, estar categorizada con **Tags** y YAML frontmatter, y estar conectada al resto del cerebro usando **enlaces bidireccionales (`[[ ]]`)**.
- Los índices o Mapas de Contenido (MOCs) en `brain/00_MOCs/` deben actualizarse constantemente para que el grafo de conocimiento nunca tenga notas "huérfanas".

## 6. Ubicación y Estructura de Subagentes
- A partir de ahora, la ubicación estándar para la definición e instrucciones de todos los subagentes dentro del espacio de trabajo será:
  - **`.agents/agents/<name>/agent.md`** (donde `<name>` es el identificador o nombre del subagente).
- Cada subagente debe contar con su archivo `agent.md` correspondiente describiendo su rol, responsabilidades, herramientas asignadas y flujos de trabajo específicos.



## 7. Fuente de Verdad del Esquema: Data-First (SQL versionado)
- A partir del 2026-10-11 la **única fuente de verdad del esquema** son las migraciones SQL en **`infra/database/migrations/*.sql`** (DbUp, forward-only, inmutables). Está fuera de `src/` porque `src/` está en `.gitignore`.
- **EF Core solo mapea**; no se generan migraciones con `dotnet ef`. La migración EF `Initial` se descarta.
- Los archivos **DBML son artefactos generados** desde la base real (`db2dbml`); nunca se editan a mano ni se usan para decidir el esquema.
- Toda tabla debe cumplir las convenciones de `infra/database/conventions.md` y el *Definition of Done* de `brain/01_Architecture/Roadmap_Data_First.md`.
- Las reglas de los puntos 1 y 2 aplican igual, sustituyendo "migración EF" por "migración SQL". Decisiones registradas como ADR en `brain/01_Architecture/ADR/`.
