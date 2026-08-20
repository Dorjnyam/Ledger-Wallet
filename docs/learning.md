# Learning Log — General Ledger & Wallet Service

**Status:** Living Document  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Project Charter](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/00-project-charter.md)

---

## 1. Objective / Purpose

This document captures **technical concepts learned**, **unresolved questions**, and **knowledge gaps** encountered during implementation. Following the **70% Rule** — when you hit something you don't fully understand, log it here and keep moving. Come back to research later.

---

## 2. Concepts Learned

| Date | Concept | Summary | Source |
|---|---|---|---|
| 2026-08-20 | Double-entry bookkeeping | Every transaction has ≥2 entries; total debits must equal total credits | Assignment research |
| 2026-08-20 | The five account types | Asset, Liability, Equity, Revenue, Expense — each has different debit/credit normal balance rules | Accounting textbooks |
| 2026-08-20 | Normal balance | Assets & Expenses increase with debits; Liabilities, Equity & Revenue increase with credits | Research |
| 2026-08-20 | Historical balance | Balance of an account as of a specific point in time, computed by filtering entries up to that timestamp | Assignment brief |
| 2026-08-20 | Idempotency in payment systems | Same request with same key must produce same result without side effects — critical to prevent double-charging | Assignment brief + industry practice |
| | | | |

---

## 3. Unresolved Questions / To Research

| # | Question | Priority | Status |
|---|---|---|---|
| Q-001 | How does `SELECT … FOR UPDATE` behave with connection pool contention under high concurrency? | Medium | Open |
| Q-002 | Should reversal transactions reference the original transaction ID? What's the standard pattern? | Medium | Open |
| Q-003 | How to handle timezone edge cases in historical balance queries? (UTC vs. local time) | Low | Open |
| Q-004 | What is the `SKIP LOCKED` optimization and when would it apply to this system? | Low | Open |
| Q-005 | How do production ledger systems handle multi-currency with exchange rates? | Low | Open (v2) |

---

## 4. Mistakes Made & Lessons

| Date | Mistake | Lesson Learned |
|---|---|---|
| | *(Log mistakes here as they happen during implementation)* | |

> [!TIP]
> This document is for **you**. Be honest about what you don't know. It's far better to say "I logged this as a question and researched it" at the review than to pretend understanding and get caught.
