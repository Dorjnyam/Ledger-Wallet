# Test Plan — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Functional Requirements](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-2-requirements/functional-requirements.md), [Non-Functional Requirements](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-2-requirements/non-functional-requirements.md)

---

## 1. Objective / Purpose

Define the **testing strategy** for the Ledger & Wallet Service. Every test proves a specific invariant — these are not token tests to boost coverage, but **correctness proofs**.

---

## 2. Testing Approach

| Level | Tool | Scope | Runs With |
|---|---|---|---|
| Unit Tests | `testing` (stdlib) | Domain logic, service methods | `go test ./...` |
| Integration Tests | `testing` + test DB | Store layer against real PostgreSQL | `go test -tags=integration ./...` |
| Concurrency Tests | `testing` + goroutines | Race condition prevention | `go test -race ./...` |
| Manual / API Tests | `curl` + `docker compose` | Full end-to-end flow | Manual demo |

---

## 3. Test Matrix — Invariant Proofs

### Ledger Invariants

| Test ID | Invariant | Test Description | Expected Result |
|---|---|---|---|
| T-001 | Debits = Credits | Post a transaction where debits ≠ credits | Transaction rejected, no entries written |
| T-002 | Debits = Credits | Post a balanced transaction | Transaction accepted, entries persisted |
| T-003 | Balance derivation | Create entries, query balance | Balance = SUM(debits) - SUM(credits) for Asset |
| T-004 | Balance derivation | Query balance on account with zero entries | Balance = 0 |
| T-005 | Immutability | Attempt to UPDATE an entry via raw SQL | Database rejects with permission error |
| T-006 | Immutability | Attempt to DELETE an entry via raw SQL | Database rejects with permission error |
| T-007 | Atomicity | Post transaction with one valid and one invalid entry | Entire transaction rejected, no partial writes |

### Historical Balance

| Test ID | Invariant | Test Description | Expected Result |
|---|---|---|---|
| T-010 | Historical accuracy | Post entries at T1 and T2; query balance at T1 | Only T1 entries counted |
| T-011 | Historical accuracy | Query balance at time before any entries | Balance = 0 |
| T-012 | Historical accuracy | Query balance at current time | Matches current balance query |

### Wallet Invariants

| Test ID | Invariant | Test Description | Expected Result |
|---|---|---|---|
| T-020 | No overdraft | Withdraw more than wallet balance | 422 error, no entries written |
| T-021 | No overdraft | Withdraw exactly the wallet balance | Succeeds, balance = 0 |
| T-022 | Deposit correctness | Deposit $100 into wallet | Cash debited, Wallet credited, balance = $100 |
| T-023 | Transfer correctness | Transfer $50 from Alice to Bob | Alice -$50, Bob +$50, total unchanged |
| T-024 | Transfer self-reject | Transfer from wallet to itself | 400 error |

### Idempotency

| Test ID | Invariant | Test Description | Expected Result |
|---|---|---|---|
| T-030 | No duplicates | Submit deposit with same idempotency_key twice | Second request returns existing result, entry count unchanged |
| T-031 | Key uniqueness | Submit different operations with same key | Second rejected or returns first result |

### Concurrency

| Test ID | Invariant | Test Description | Expected Result |
|---|---|---|---|
| T-040 | Race safety | 100 goroutines withdraw $1 from wallet with $50 | Exactly 50 succeed, 50 fail; final balance = $0 |
| T-041 | Race safety | 2 goroutines withdraw $80 from wallet with $100 | Exactly 1 succeeds, 1 fails; final balance = $20 |
| T-042 | No deadlocks | Concurrent transfers: Alice→Bob and Bob→Alice | Both complete (one may fail on balance), no deadlock |

---

## 4. Manual Testing Checklist (Demo Flow)

Run after `docker compose up`:

- [ ] Create a Cash account (Asset, USD)
- [ ] Create Alice wallet (Liability, USD)
- [ ] Create Bob wallet (Liability, USD)
- [ ] Deposit $100 into Alice's wallet
- [ ] Verify Alice's balance = 10000 (cents)
- [ ] Verify Cash balance = 10000 (cents)
- [ ] Transfer $30 from Alice to Bob
- [ ] Verify Alice = 7000, Bob = 3000
- [ ] Withdraw $20 from Alice
- [ ] Verify Alice = 5000, Cash = 8000
- [ ] Attempt to withdraw $9999 from Alice → expect 422
- [ ] Verify Alice balance unchanged at 5000
- [ ] Re-submit the deposit with same idempotency_key → expect same result
- [ ] Query Alice's historical balance at a past timestamp → correct snapshot
- [ ] View Alice's transaction history → shows all transactions

---

## 5. Running Tests

```bash
# Run all unit tests
go test ./...

# Run with race detector
go test -race ./...

# Run integration tests (requires running Postgres)
go test -tags=integration ./...

# Run specific test
go test -run TestConcurrentWithdrawals ./internal/wallet/...
```
