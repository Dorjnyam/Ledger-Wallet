# Environment Setup — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Deployment Runbook](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-7-deployment/deployment-runbook.md), [Architecture](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-4-design/architecture.md)

---

## 1. Objective / Purpose

Document all **environment variables** and **configuration** needed to run the Ledger & Wallet Service locally and in production.

---

## 2. Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `DATABASE_URL` | Yes | — | PostgreSQL connection string (e.g., `postgres://user:pass@localhost:5432/ledger?sslmode=disable`) |
| `DB_HOST` | No | `localhost` | Database host (used if `DATABASE_URL` is not set) |
| `DB_PORT` | No | `5432` | Database port |
| `DB_USER` | No | `postgres` | Database user |
| `DB_PASSWORD` | No | `postgres` | Database password |
| `DB_NAME` | No | `ledger` | Database name |
| `DB_SSLMODE` | No | `disable` | SSL mode for database connection |
| `SERVER_PORT` | No | `8080` | HTTP server port |
| `LOG_LEVEL` | No | `info` | Log level: debug, info, warn, error |
| `DB_MAX_CONNS` | No | `25` | Maximum database connection pool size |

---

## 3. Docker Compose Environment

The `docker-compose.yml` sets these automatically:

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: ledger
    ports:
      - "5432:5432"

  app:
    build: .
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/ledger?sslmode=disable
      SERVER_PORT: "8080"
    ports:
      - "8080:8080"
    depends_on:
      - db
```

---

## 4. Local Development (Without Docker)

1. Install PostgreSQL 16+
2. Create database:
   ```bash
   createdb ledger
   ```
3. Set environment variables:
   ```bash
   export DATABASE_URL="postgres://postgres:postgres@localhost:5432/ledger?sslmode=disable"
   export SERVER_PORT="8080"
   ```
4. Run migrations:
   ```bash
   go run cmd/server/main.go  # migrations run on startup
   ```

---

## 5. Prerequisites

| Tool | Minimum Version | Installation |
|---|---|---|
| Go | 1.22+ | [go.dev/dl](https://go.dev/dl/) |
| PostgreSQL | 16+ | `brew install postgresql` / `apt install postgresql` / Docker |
| Docker | 24+ | [docker.com](https://docker.com) |
| Docker Compose | v2+ | Bundled with Docker Desktop |

> [!WARNING]
> **Never commit real credentials.** The values above are for local development only. In production, use secrets management (e.g., Vault, cloud provider secrets).
