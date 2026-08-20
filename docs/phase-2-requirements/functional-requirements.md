# Functional Requirements — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Project Charter](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/00-project-charter.md), [Non-Functional Requirements](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-2-requirements/non-functional-requirements.md)

---

## 1. Objective / Purpose

Define every **functional requirement** for the Ledger & Wallet Service, extracted from the assignment brief. Each requirement is assigned a unique ID, priority, and acceptance criterion.

---

## 2. Account System

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| FR-001 | System supports five account types: Asset, Liability, Equity, Revenue, Expense | High | Account creation succeeds for each type; rejects unknown types |
| FR-002 | Each account has a unique ID, name, type, and currency | High | Account fields are stored and retrievable via API |
| FR-003 | Wallets are accounts (typically Liability type from platform perspective) | High | Wallet operations work on Liability accounts |
| FR-004 | Accounts can be created via API | High | `POST /v1/accounts` returns 201 with account details |

---

## 3. Double-Entry Transactions

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| FR-010 | A transaction consists of two or more entries (postings) | High | API accepts transactions with ≥2 entries |
| FR-011 | Each entry debits or credits one account | High | Entry has `account_id`, `direction` (debit/credit), and `amount` |
| FR-012 | Total debits must equal total credits per transaction | High | Transaction with unbalanced entries is rejected with 422 |
| FR-013 | Invalid transactions are rejected atomically — no partial writes | High | If any entry is invalid, no entries from that transaction are persisted |
| FR-014 | Transactions can be posted via API | High | `POST /v1/transactions` returns 201 with transaction + entry details |

---

## 4. Immutability

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| FR-020 | Ledger entries can never be updated or deleted | High | No API endpoint for UPDATE/DELETE; DB `REVOKE` enforces this |
| FR-021 | Corrections are made by posting a reversal transaction | High | Reversal creates new entries that cancel out the original |

---

## 5. Balance Calculation

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| FR-030 | Current balance is computed from sum of entries (not stored directly) | High | `GET /v1/accounts/:id/balance` returns balance derived from entries |
| FR-031 | Balance follows debit/credit rules per account type | High | Asset/Expense: debits - credits; Liability/Equity/Revenue: credits - debits |
| FR-032 | Historical balance query: given account + timestamp, return balance at that moment | High | `GET /v1/accounts/:id/balance?at=<ISO8601>` returns correct historical balance |

---

## 6. Wallet Operations

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| FR-040 | Deposit into a wallet | High | `POST /v1/wallets/deposit` creates correct double-entry transaction (debit cash, credit wallet) |
| FR-041 | Withdraw from a wallet | High | `POST /v1/wallets/withdraw` creates correct double-entry transaction (debit wallet, credit cash) |
| FR-042 | Transfer between two wallets | High | `POST /v1/wallets/transfer` creates correct double-entry transaction (debit source, credit destination) |
| FR-043 | Withdrawal/transfer fails if it would make wallet balance negative | High | Returns 422 with clear error message; no entries written |

---

## 7. Idempotency

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| FR-050 | Every wallet operation accepts an idempotency key | High | Request body includes `idempotency_key` field |
| FR-051 | Duplicate idempotency key returns existing result, no new transaction | High | Second request with same key returns 200 with original transaction |

---

## 8. Transaction History

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| FR-060 | View transaction history for an account | Medium | `GET /v1/accounts/:id/transactions` returns list of transactions involving that account |
