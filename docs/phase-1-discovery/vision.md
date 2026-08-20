# Vision — General Ledger & Wallet Service

**Status:** Approved  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Project Charter](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/00-project-charter.md)

---

## 1. Objective / Purpose

Define the **purpose and vision** of the General Ledger & Wallet Service — what it is, what problem it solves, and why it matters.

---

## 2. Vision Statement

> Build a **production-quality double-entry general ledger engine** that proves mastery of financial accounting fundamentals, concurrent systems design, and Go software engineering. Every wallet balance is a **mathematical fact** derived from immutable ledger entries — never a mutable number sitting in a column.

---

## 3. Why This Matters

### In the Real World
Every fintech company, bank, payment processor, and marketplace runs some form of a double-entry ledger. Understanding how money moves through accounts — and how to ensure it **never** gets lost or duplicated — is the foundational skill for building financial software.

### For This Assignment
This is not a CRUD app. It tests:

| Concept | What It Proves |
|---|---|
| **Double-entry bookkeeping** | You understand the accounting model, not just API plumbing |
| **Immutability** | You can design systems where data integrity is enforced by architecture, not discipline |
| **Concurrency** | You can reason about race conditions and prevent them with database primitives |
| **Idempotency** | You understand payment system design patterns |
| **Historical balance** | You can compute point-in-time state from an append-only log |

---

## 4. What Success Looks Like

1. A running service that you can demo live with `docker compose up`
2. A test suite that **proves** the invariants hold (not just tests that pass)
3. A README and docs folder that a reviewer can read and immediately understand your design
4. A commit history that shows thoughtful, incremental development — not one massive commit
5. The ability to explain every decision when asked "Why?"
