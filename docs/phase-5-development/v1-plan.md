# v1 Development Plan — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Roadmap](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/roadmap.md), [Architecture](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-4-design/architecture.md)

---

## 1. Objective / Purpose

Tactical execution plan for building v1 of the Ledger & Wallet Service. Each day maps to a focused layer of the architecture. Commits should be incremental and descriptive.

---

## 2. Day-by-Day Breakdown

### Day 1 — Foundation & Domain

| Task | File(s) | Commit Message Pattern |
|---|---|---|
| Init Go module | `go.mod` | `chore: init go module` |
| Define domain types | `internal/domain/*.go` | `feat(domain): add Account, Transaction, Entry types` |
| Define account type constants | `internal/domain/account.go` | `feat(domain): add AccountType enum and validation` |
| Define money type | `internal/domain/money.go` | `feat(domain): add Amount type (int64 minor units)` |
| Write SQL migrations | `migrations/*.sql` | `feat(migrations): create accounts/transactions/entries tables` |
| Write Dockerfile | `Dockerfile` | `chore: add Dockerfile for Go service` |
| Write docker-compose.yml | `docker-compose.yml` | `chore: add docker-compose with Postgres` |

**End of Day 1 checkpoint:** `docker compose up` starts Postgres + Go binary (even if no API routes yet). All 3 tables exist.

---

### Day 2 — Store & Ledger Core

| Task | File(s) | Commit Message Pattern |
|---|---|---|
| Implement store constructor + pool | `internal/store/store.go` | `feat(store): add Store with pgxpool setup` |
| Implement account store | `internal/store/account_store.go` | `feat(store): add CreateAccount, GetAccount` |
| Implement transaction store | `internal/store/transaction_store.go` | `feat(store): add CreateTransaction with entries` |
| Implement balance store | `internal/store/balance_store.go` | `feat(store): add current and historical balance queries` |
| Implement ledger service | `internal/ledger/service.go` | `feat(ledger): add PostTransaction with debit=credit validation` |
| Write ledger tests | `internal/ledger/service_test.go` | `test(ledger): verify balanced transactions and rejection of unbalanced` |

**End of Day 2 checkpoint:** Ledger service can create accounts, post transactions, and compute balances. Tests pass.

---

### Day 3 — Wallet Layer

| Task | File(s) | Commit Message Pattern |
|---|---|---|
| Implement wallet service | `internal/wallet/service.go` | `feat(wallet): add Deposit, Withdraw, Transfer` |
| Add concurrency locking | `internal/store/balance_store.go` | `feat(store): add SELECT FOR UPDATE balance lock` |
| Add idempotency check | `internal/store/transaction_store.go` | `feat(store): add idempotency key deduplication` |
| Write wallet tests | `internal/wallet/service_test.go` | `test(wallet): verify no overdraft, idempotent retries` |
| Write concurrency tests | `internal/wallet/service_test.go` | `test(wallet): verify concurrent withdrawal safety` |

**End of Day 3 checkpoint:** Wallet operations work correctly. Concurrency and idempotency tests pass.

---

### Day 4 — HTTP API

| Task | File(s) | Commit Message Pattern |
|---|---|---|
| Set up chi router | `internal/api/router.go` | `feat(api): add chi router with middleware` |
| Implement account handlers | `internal/api/accounts_handler.go` | `feat(api): add POST/GET account endpoints` |
| Implement transaction handler | `internal/api/transactions_handler.go` | `feat(api): add POST transaction endpoint` |
| Implement wallet handlers | `internal/api/wallet_handler.go` | `feat(api): add deposit/withdraw/transfer endpoints` |
| Implement balance handler | `internal/api/balance_handler.go` | `feat(api): add current/historical balance endpoints` |
| Wire main.go | `cmd/server/main.go` | `feat(cmd): wire dependencies and start server` |
| Add request validation | `internal/api/*.go` | `feat(api): add input validation and error responses` |

**End of Day 4 checkpoint:** All API endpoints respond correctly. Can demo full flow with `curl`.

---

### Day 5 — Documentation, Polish & Final Testing

| Task | File(s) | Commit Message Pattern |
|---|---|---|
| Write README | `README.md` | `docs: add README with run instructions and design decisions` |
| Review and update docs/ | `docs/**/*.md` | `docs: finalize phase documentation` |
| Write integration tests | `*_test.go` | `test: add end-to-end integration tests` |
| Review commit history | — | Squash/reword if needed for clean history |
| Test full demo flow | — | Manual: docker compose up → curl walkthrough |

**End of Day 5 checkpoint:** All deliverables complete. Ready for review.

---

## 3. Commit Strategy

- **Target:** 20+ commits with clean messages
- **Format:** Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `chore:`)
- **Scope:** Use package names as scope (e.g., `feat(ledger):`, `test(wallet):`)
- **Principle:** Each commit should be a logical, buildable unit. Never commit broken code to `main`.

---

## 4. Risk Mitigation

| Risk | Likelihood | Mitigation |
|---|---|---|
| Concurrency implementation takes longer than expected | Medium | Start with simplest correct approach (`SELECT FOR UPDATE`); don't prematurely optimize |
| Historical balance edge cases (timezone, boundary conditions) | Medium | Use `TIMESTAMPTZ` everywhere; test with explicit timestamps in UTC |
| Docker/environment setup issues | Low | Test `docker compose up` early on Day 1; don't save it for Day 5 |
| Running out of time on docs | Medium | Write docs incrementally (not all on Day 5); start README on Day 1 |
