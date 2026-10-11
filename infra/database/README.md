# Nexus Database (Data-First)

**Fuente de verdad del esquema:** `migrations/*.sql` (forward-only, aplicadas con DbUp).
EF Core solo mapea. El DBML se **genera** desde la BD:
`npx -p @dbml/cli db2dbml postgres "<conn>" -o brain/01_Architecture/Nexus.dbml`

> `src/` está en `.gitignore` del repo raíz, por eso la BD vive en `infra/database/` (versionada).

Ver [conventions.md](conventions.md) y `brain/01_Architecture/Roadmap_Data_First.md`.

## Levantar entorno
```bash
cp infra/database/.env.example infra/database/.env   # editar password
docker compose -f infra/database/docker-compose.yml --env-file infra/database/.env up -d
```
