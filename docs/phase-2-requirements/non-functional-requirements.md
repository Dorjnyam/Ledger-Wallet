# Non-Functional Requirements — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Functional Requirements](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-2-requirements/functional-requirements.md), [Architecture Evaluation](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/architecture-evaluation.md)

---

## 1. Objective / Purpose

Define the **non-functional requirements** (quality attributes) for the Ledger & Wallet Service. These describe *how well* the system must perform, not *what* it does.

---

## 2. Non-Functional Requirements

### Correctness (NFR-1xx)

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-100 | Debits = Credits invariant must hold on every transaction | 100% — zero tolerance | Unit test: reject unbalanced transactions |
| NFR-101 | No negative wallet balances under any circumstance | 100% — no overdraft | Unit test + concurrent goroutine test |
| NFR-102 | Idempotent retry produces identical result, no side effects | 100% | Unit test: same key → same result, entry count unchanged |
| NFR-103 | Historical balance is correct at any given timestamp | 100% | Unit test: insert at T1, T2; query at T1; assert T1-only entries counted |

### Data Integrity (NFR-2xx)

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-200 | Ledger entries are immutable — no UPDATE, no DELETE | Enforced at DB level | `REVOKE UPDATE, DELETE ON entries`; attempt UPDATE → error |
| NFR-201 | Transactions are immutable | Enforced at DB level | `REVOKE UPDATE, DELETE ON transactions` |
| NFR-202 | No partial writes — all entries in a transaction succeed or all fail | Atomic DB transactions | Unit test: one bad entry → entire transaction rejected |

### Concurrency (NFR-3xx)

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-300 | Concurrent withdrawals from same wallet — only one succeeds if funds cover one | Race condition prevented | Goroutine test: 100 concurrent withdrawals on balance-for-one |
| NFR-301 | No deadlocks under normal operation | Zero deadlocks | Consistent lock ordering (lock accounts by ID ascending) |

### Performance (NFR-4xx)

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-400 | API response time for balance queries | < 100ms for accounts with < 10K entries | Manual testing |
| NFR-401 | API response time for posting transactions | < 200ms under normal load | Manual testing |

> [!NOTE]
> Performance targets are soft guidelines for this assignment, not SLA commitments. Correctness always takes priority over speed.

### Operability (NFR-5xx)

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-500 | Service runs via `docker compose up` with zero manual steps | One command to start | Manual test: clone repo → `docker compose up` → API responds |
| NFR-501 | All configuration via environment variables | No hardcoded secrets | Check `.env` / docker-compose.yml |
| NFR-502 | Database migrations run automatically on startup | Zero manual SQL | Migrations embedded in startup sequence |

### Code Quality (NFR-6xx)

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-600 | Test suite passes with `go test ./...` | 100% pass rate | CI or manual `go test` |
| NFR-601 | Clean commit history with 20+ commits | Incremental, descriptive commits | `git log --oneline` |
| NFR-602 | Code comments where extra explanation is needed | Judgment-based | Code review |
| NFR-603 | README explains how to run, API endpoints, and design decisions | Comprehensive | Manual review |
