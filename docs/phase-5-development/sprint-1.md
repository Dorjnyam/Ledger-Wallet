# Sprint 1 — Foundation & Core Ledger

**Status:** Not Started  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [v1 Plan](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-5-development/v1-plan.md), [Architecture](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-4-design/architecture.md)

---

## 1. Objective / Purpose

First sprint covering Days 1–2 of the v1 plan. Establishes the project foundation, domain types, database schema, and the core ledger engine.

---

## 2. Task Checklist

### Day 1 — Foundation

- [ ] Initialize Go module (`go mod init`)
- [ ] Add dependencies: `pgx/v5`, `chi`, `golang-migrate`
- [ ] Create `internal/domain/account.go` — Account struct, AccountType constants
- [ ] Create `internal/domain/transaction.go` — Transaction, Entry structs
- [ ] Create `internal/domain/money.go` — Amount type (int64)
- [ ] Create migration: `000001_create_accounts.up.sql` / `.down.sql`
- [ ] Create migration: `000002_create_transactions.up.sql` / `.down.sql`
- [ ] Create migration: `000003_create_entries.up.sql` / `.down.sql`
- [ ] Create `Dockerfile`
- [ ] Create `docker-compose.yml` (Postgres + Go app)
- [ ] Verify: `docker compose up` starts successfully

### Day 2 — Ledger Core

- [ ] Create `internal/store/store.go` — Store struct with `pgxpool.Pool`
- [ ] Create `internal/store/account_store.go` — `CreateAccount`, `GetAccountByID`
- [ ] Create `internal/store/transaction_store.go` — `CreateTransactionWithEntries`
- [ ] Create `internal/store/balance_store.go` — `GetBalance`, `GetHistoricalBalance`
- [ ] Create `internal/ledger/service.go` — `PostTransaction` with debit=credit check
- [ ] Create `internal/ledger/service_test.go` — invariant tests
- [ ] Verify: `go test ./internal/ledger/...` passes

---

## 3. Acceptance Criteria

- [ ] All domain types compile and have clear field documentation
- [ ] Migrations create all 3 tables with correct constraints
- [ ] `REVOKE UPDATE, DELETE ON entries` is in migration SQL
- [ ] Ledger service rejects unbalanced transactions
- [ ] Ledger service correctly computes balance from entries
- [ ] All tests pass

---

## 4. Blockers & Notes

| Item | Status |
|---|---|
| *(Log blockers here as they arise)* | |
