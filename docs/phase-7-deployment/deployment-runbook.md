# Deployment Runbook — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Environment Setup](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-7-deployment/environment-setup.md), [Architecture](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-4-design/architecture.md)

---

## 1. Objective / Purpose

Step-by-step guide for **building, running, and deploying** the Ledger & Wallet Service. This is the runbook that the reviewer (or any developer) follows to get the system up and running.

---

## 2. Quick Start (Docker Compose — Recommended)

```bash
# 1. Clone the repository
git clone <repo-url>
cd ledger-wallet

# 2. Start everything (Postgres + Go service)
docker compose up --build

# 3. Wait for "Server started on :8080" in logs

# 4. Test the API
curl http://localhost:8080/v1/accounts
```

**That's it.** Docker Compose handles:
- Starting PostgreSQL
- Running database migrations
- Building and starting the Go service

---

## 3. Manual Deployment (Without Docker)

### Step 1: Database Setup

```bash
# Start PostgreSQL (if not already running)
pg_ctl start

# Create the database
createdb ledger

# Set connection string
export DATABASE_URL="postgres://postgres:postgres@localhost:5432/ledger?sslmode=disable"
```

### Step 2: Run Migrations

Migrations run automatically on application startup. Alternatively, run them manually:

```bash
go run cmd/migrate/main.go up
```

### Step 3: Build & Run

```bash
# Build the binary
go build -o ledger-wallet ./cmd/server

# Run it
./ledger-wallet
```

### Step 4: Verify

```bash
# Health check
curl http://localhost:8080/v1/accounts

# Create a test account
curl -X POST http://localhost:8080/v1/accounts \
  -H "Content-Type: application/json" \
  -d '{"name": "Cash", "type": "asset", "currency": "USD"}'
```

---

## 4. Dockerfile

```dockerfile
# Multi-stage build for minimal image size
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /ledger-wallet ./cmd/server

FROM alpine:3.19
RUN apk --no-cache add ca-certificates
COPY --from=builder /ledger-wallet /ledger-wallet
COPY migrations/ /migrations/
EXPOSE 8080
ENTRYPOINT ["/ledger-wallet"]
```

---

## 5. Database Migrations

Migrations are in `migrations/` and follow the naming convention:

```
000001_create_accounts.up.sql
000001_create_accounts.down.sql
000002_create_transactions.up.sql
000002_create_transactions.down.sql
000003_create_entries.up.sql
000003_create_entries.down.sql
```

> [!IMPORTANT]
> The `000003_create_entries.up.sql` migration includes `REVOKE UPDATE, DELETE ON entries FROM PUBLIC`. This enforces immutability at the database level.

### Rolling Back

```bash
# Roll back the last migration
go run cmd/migrate/main.go down 1
```

---

## 6. Demo Flow (For Review)

Use this exact sequence to demonstrate the system at the review:

```bash
# 1. Start the system
docker compose up --build -d

# 2. Create Cash account (Asset)
curl -s -X POST http://localhost:8080/v1/accounts \
  -H "Content-Type: application/json" \
  -d '{"name": "Cash", "type": "asset", "currency": "USD"}' | jq

# 3. Create Alice's wallet (Liability)
curl -s -X POST http://localhost:8080/v1/accounts \
  -H "Content-Type: application/json" \
  -d '{"name": "Wallet:Alice", "type": "liability", "currency": "USD"}' | jq

# 4. Deposit $100 into Alice's wallet
curl -s -X POST http://localhost:8080/v1/wallets/deposit \
  -H "Content-Type: application/json" \
  -d '{"wallet_id": "<alice-uuid>", "amount": 10000, "idempotency_key": "demo-dep-1"}' | jq

# 5. Check Alice's balance (should be 10000)
curl -s http://localhost:8080/v1/accounts/<alice-uuid>/balance | jq

# 6. Attempt overdraft (should fail with 422)
curl -s -X POST http://localhost:8080/v1/wallets/withdraw \
  -H "Content-Type: application/json" \
  -d '{"wallet_id": "<alice-uuid>", "amount": 99999, "idempotency_key": "demo-wd-fail"}' | jq

# 7. Retry deposit with same idempotency key (should return same result)
curl -s -X POST http://localhost:8080/v1/wallets/deposit \
  -H "Content-Type: application/json" \
  -d '{"wallet_id": "<alice-uuid>", "amount": 10000, "idempotency_key": "demo-dep-1"}' | jq

# 8. Query historical balance
curl -s "http://localhost:8080/v1/accounts/<alice-uuid>/balance?at=2026-08-20T00:00:00Z" | jq
```

> [!TIP]
> Replace `<alice-uuid>` with the actual UUID returned from step 3. Use `jq` for pretty-printed JSON output.

---

## 7. Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `connection refused` on port 8080 | App not running or still starting | Wait a few seconds; check `docker compose logs app` |
| `connection refused` on port 5432 | PostgreSQL not running | `docker compose logs db`; ensure port 5432 is free |
| `relation "accounts" does not exist` | Migrations didn't run | Check startup logs for migration output |
| `permission denied for table entries` | Trying to UPDATE/DELETE entries | This is by design — entries are immutable |
