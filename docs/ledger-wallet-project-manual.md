# Ledger & Wallet Service — Project Manual

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Project Charter](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/00-project-charter.md), [Roadmap](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/roadmap.md)

---

## 1. Objective / Purpose

This is the **master reference document** for the General Ledger & Wallet Service project. It summarizes the full Software Development Life Cycle (SDLC) from Discovery through Deployment, providing a phase-by-phase checklist and linking to detailed documents in each phase folder.

---

## 2. Project Overview

| Field | Value |
|---|---|
| **What** | A double-entry general ledger engine with a wallet API layer |
| **Why** | Onboarding assignment — demonstrate understanding of financial accounting software, Go, PostgreSQL, and production concerns (concurrency, idempotency, immutability) |
| **Tech Stack** | Go + PostgreSQL + `chi` + `pgx/v5` + Docker |
| **Timeline** | 5 working days |
| **Success Criteria** | Correct ledger invariants, passing test suite, clean commit history, explainable design |

---

## 3. Phase-by-Phase Checklist

### Phase 1: Discovery
- [ ] Research double-entry bookkeeping fundamentals
- [ ] Research the five account types (Asset, Liability, Equity, Revenue, Expense)
- [ ] Research historical balance concepts
- [ ] Research debit/credit rules per account type
- [ ] Identify all stakeholders and users of the system
- [ ] Document assumptions and constraints

**Docs:** [phase-1-discovery/](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-1-discovery/)

### Phase 2: Requirements
- [ ] Extract all functional requirements from the assignment brief
- [ ] Define non-functional requirements (performance, correctness, concurrency)
- [ ] Map each requirement to a testable acceptance criterion

**Docs:** [phase-2-requirements/](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-2-requirements/)

### Phase 3: Analysis
- [ ] Design the data model (accounts, transactions, entries)
- [ ] Map use cases (deposit, withdraw, transfer, balance queries)
- [ ] Define entity relationships and constraints
- [ ] Document the accounting equation and how it maps to code

**Docs:** [phase-3-analysis/](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-3-analysis/)

### Phase 4: Design
- [ ] Define system architecture (layers, dependencies, data flow)
- [ ] Specify all API endpoints (REST)
- [ ] Document concurrency and idempotency strategies
- [ ] Document the accounting concepts mapping (debit/credit per operation)

**Docs:** [phase-4-design/](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-4-design/)

### Phase 5: Development
- [ ] Implement in sprints with incremental commits
- [ ] Follow the v1 development plan
- [ ] Log progress and blockers

**Docs:** [phase-5-development/](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-5-development/)

### Phase 6: Testing
- [ ] Write unit tests for ledger invariants
- [ ] Write concurrency tests
- [ ] Write idempotency tests
- [ ] Write historical balance tests
- [ ] Track bugs and resolutions

**Docs:** [phase-6-testing/](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-6-testing/)

### Phase 7: Deployment
- [ ] Write `docker-compose.yml` for Postgres + Go service
- [ ] Write `Dockerfile` for the Go service
- [ ] Document environment variables
- [ ] Write deployment runbook

**Docs:** [phase-7-deployment/](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-7-deployment/)

---

## 4. Key Principles

1. **Correctness Over Features**: A small system that never breaks `Debits = Credits` beats a large system that sometimes does.
2. **Immutability by Design**: Ledger entries are never updated or deleted — only reversed.
3. **Traceability**: Every wallet balance is derived from summing entries, never stored directly.
4. **Defensibility**: Every design decision has a documented "Why" that you can explain at review.

---

## 5. Review Preparation

At the end of the 5-day period, be ready to:

1. **Demo** the running service via `docker compose up` + `curl`
2. **Walk through** the codebase layer by layer
3. **Explain** concurrency strategy (row locking, isolation levels)
4. **Explain** idempotency mechanism (duplicate key handling)
5. **Show** the test suite and what invariants it proves

> [!TIP]
> Practice the demo flow before the review. Run through create account → deposit → transfer → withdraw → check balance → check historical balance → attempt overdraft → attempt duplicate idempotency key.
