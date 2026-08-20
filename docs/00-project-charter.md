# Project Charter — General Ledger & Wallet Service

**Status:** Approved  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Project Manual](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/ledger-wallet-project-manual.md), [Roadmap](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/roadmap.md)

---

## 1. Project Type & Context

| Field | Value |
|---|---|
| **Project Type** | Onboarding Assignment (Stage 2 — Coding) |
| **Client / Employer** | Internal evaluation — no external client |
| **Requirement Source** | Assignment brief provided by evaluator |
| **Deliverable Deadline** | 5 working days from start |
| **Review Format** | Live demo + code walkthrough + Q&A on concurrency & idempotency |

> [!IMPORTANT]
> This is an **evaluation project**. Correctness of ledger invariants matters more than feature count. A small system that **never** breaks `Debits = Credits` is worth far more than a large system that sometimes does.

---

## 2. Problem Statement

Build a **double-entry general ledger engine** with a **wallet layer** on top of it. Every wallet balance must be derived from ledger entries — never stored as a standalone number that gets updated directly. The system must be correct, concurrent-safe, idempotent, and immutable.

---

## 3. Core Non-Negotiable Constraints

These are **hard requirements** from the assignment brief that cannot be changed:

| # | Constraint | Rationale |
|---|---|---|
| C-1 | **Language: Go** (latest stable) | Assignment requirement |
| C-2 | **Storage: PostgreSQL** | Assignment requirement |
| C-3 | **No heavy ORM** — raw SQL or lightweight query layer | Ledger logic must be visible, not hidden behind abstraction |
| C-4 | **Never use `float64` for money** | Floating-point arithmetic is fundamentally broken for financial math |
| C-5 | **Immutable ledger** — no UPDATE or DELETE on entries | Corrections via reversal transactions only |
| C-6 | **Debits = Credits** on every transaction, enforced atomically | The bedrock invariant of double-entry bookkeeping |
| C-7 | **No negative wallet balances** (no overdraft) | Assignment requirement |
| C-8 | **Idempotency keys** on all wallet operations | Critical for payment systems — no duplicate transactions |
| C-9 | **Concurrency safety** — concurrent withdrawals must not both succeed if funds cover only one | Must implement and defend the chosen mechanism |

---

## 4. Technology Decisions

| Decision | Choice | Why |
|---|---|---|
| HTTP Framework | `chi` (stdlib-compatible router) | Minimal dependencies, idiomatic Go, easy to test |
| Database Driver | `pgx/v5` | Pure Go PostgreSQL driver, production-grade, supports `pgxpool` |
| Migrations | `golang-migrate/migrate` | SQL-file-based, no magic, version-controlled schema |
| Money Representation | `int64` minor units (cents) | Exact integer arithmetic; documented in README |
| Concurrency Strategy | `SELECT … FOR UPDATE` row-level locking | Simple, correct, easy to explain at review |
| API Style | REST + JSON | Easier to demo with `curl`; gRPC is overkill for this scope |

> [!NOTE]
> Every technology decision above is documented with a **"Why"**. At the review, be prepared to explain and defend each one.

---

## 5. Deliverables Checklist

| # | Deliverable | Status |
|---|---|---|
| D-1 | Git repository with **20+ clean commits** showing working process | Not Started |
| D-2 | README: how to run (`docker compose`), API endpoints, design decisions | Not Started |
| D-3 | Test suite runnable with `go test` | Not Started |
| D-4 | Written section connecting implementation to accounting concepts | Not Started |
| D-5 | Code comments where extra explanation is needed | Not Started |

---

## 6. The 70% Rule

> Move from Discovery/Design into Coding once you understand **~70%** of the requirements. Log remaining unknowns in [learning.md](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/learning.md).

The assignment brief is well-defined. Core domain concepts (double-entry bookkeeping, account types, debit/credit rules) must be thoroughly researched **before** coding begins. But edge cases around concurrency and historical balance queries can be refined during implementation. Don't block on perfection — build, test, iterate.

---

## 7. Honesty Policy

- If any part of the codebase was AI-generated, it must be **understood and explainable** by the developer.
- Every design decision must have a documented **"Why"** — not just a **"What"**.
- If a concept is not fully understood, log it in `learning.md` rather than pretending mastery.
