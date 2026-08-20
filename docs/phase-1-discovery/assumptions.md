# Assumptions — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Project Charter](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/00-project-charter.md), [Non-Functional Requirements](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-2-requirements/non-functional-requirements.md)

---

## 1. Objective / Purpose

Document all **assumptions** being made in the design and implementation of this service. Distinguish between assumptions that have been **validated** (confirmed by the assignment brief) and those that are **unvalidated hypotheses** (our best guess).

---

## 2. Validated Assumptions (From Assignment Brief)

These are stated requirements — not guesses:

| # | Assumption | Source |
|---|---|---|
| VA-1 | The five account types are Asset, Liability, Equity, Revenue, Expense | Assignment brief |
| VA-2 | Wallets are liability accounts from the platform's perspective | Assignment brief |
| VA-3 | Total debits must equal total credits on every transaction | Assignment brief — double-entry rule |
| VA-4 | Entries are immutable — corrections via reversal only | Assignment brief |
| VA-5 | No overdraft — wallet balance cannot go negative | Assignment brief |
| VA-6 | Idempotency keys are required on wallet operations | Assignment brief |
| VA-7 | Concurrency must be handled — explain and implement a mechanism | Assignment brief |
| VA-8 | Money must not use floating point | Assignment brief |
| VA-9 | PostgreSQL is the required storage | Assignment brief |
| VA-10 | Go is the required language | Assignment brief |

---

## 3. Unvalidated Assumptions (Our Design Decisions)

These are reasonable assumptions we're making that are **not explicitly stated** in the brief:

| # | Assumption | Risk Level | Mitigation |
|---|---|---|---|
| UA-1 | Single-currency system (all amounts in one currency per account) | Low | Multi-currency deferred to v2; currency field exists on accounts for future use |
| UA-2 | `posted_at` timestamp is server-generated (not client-provided) | Low | Can be changed to accept client timestamps if needed |
| UA-3 | Historical balance is computed by filtering entries where `created_at <= timestamp` | Medium | Verify the evaluator means `created_at` vs `posted_at`; we'll use `posted_at` for business logic |
| UA-4 | The "cash" account for deposits/withdrawals is a platform-level asset account | Low | Standard accounting pattern |
| UA-5 | REST API is acceptable (vs. gRPC) | Low | Assignment says "REST or gRPC" — we chose REST |
| UA-6 | `SELECT … FOR UPDATE` is the expected concurrency mechanism | Low | Assignment says "explain and implement your chosen approach" — we'll defend this at review |
| UA-7 | Docker Compose is the preferred way to run the project | Low | Assignment says "docker compose preferred" |
| UA-8 | Transactions have exactly 2 entries for wallet operations (debit + credit) | Low | The assignment says "two or more entries" — wallet ops use exactly 2; raw transactions can have more |

---

## 4. Constraints vs. Assumptions

> [!IMPORTANT]
> **Constraints** are non-negotiable (see [Project Charter](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/00-project-charter.md), Section 3). **Assumptions** are beliefs that might be wrong. If an assumption proves incorrect during development, update this document and adjust the design — don't ignore it.
