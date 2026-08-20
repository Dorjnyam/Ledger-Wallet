# Accounting Concepts — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Data Model](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-3-analysis/data-model.md), [Functional Requirements](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-2-requirements/functional-requirements.md)

---

## 1. Objective / Purpose

This document connects the **implementation** back to the **accounting concepts**. It explains which accounts are debited and credited on each wallet operation, and **why** — using the rules of double-entry bookkeeping.

> [!IMPORTANT]
> This is a required deliverable: *"A short written section connecting your implementation back to the accounting concepts."*

---

## 2. The Five Account Types

In double-entry bookkeeping, every account is one of five types. Each type has a **normal balance** — the side (debit or credit) that increases the account:

| Account Type | Normal Balance | Increases With | Decreases With | Example |
|---|---|---|---|---|
| **Asset** | Debit | Debit | Credit | Cash, Bank Account, Equipment |
| **Liability** | Credit | Credit | Debit | Loans, Accounts Payable, **User Wallets** |
| **Equity** | Credit | Credit | Debit | Owner's Capital, Retained Earnings |
| **Revenue** | Credit | Credit | Debit | Sales, Service Fees |
| **Expense** | Debit | Debit | Credit | Rent, Salaries, Transaction Fees |

---

## 3. The Accounting Equation

```
Assets = Liabilities + Equity
```

Every valid double-entry transaction maintains this equation. Because `Total Debits = Total Credits` on every transaction, the equation can **never** become unbalanced.

---

## 4. Why Wallets Are Liability Accounts

From the **platform's perspective**, a user's wallet balance represents money the platform **owes** the user. If Alice deposits $100:
- The platform **has** $100 more cash (Asset ↑)
- The platform **owes** Alice $100 (Liability ↑)

This is exactly what a liability is: an obligation to pay. Therefore, user wallets are modeled as **Liability** accounts.

---

## 5. Debit/Credit Mapping Per Wallet Operation

### 5.1 Deposit — "Customer puts money in"

The platform receives cash and incurs a liability to the customer.

```
Debit:  Cash (Asset)        +$100    ← Platform gains cash
Credit: Wallet:Alice (Liability)  +$100    ← Platform now owes Alice $100
```

| Account | Type | Direction | Amount | Effect |
|---|---|---|---|---|
| Cash | Asset | **Debit** | 10000 | Asset increases (normal balance = debit) |
| Wallet:Alice | Liability | **Credit** | 10000 | Liability increases (normal balance = credit) |

**Verify equation:** Assets (+100) = Liabilities (+100) + Equity (0) ✅

---

### 5.2 Withdrawal — "Customer takes money out"

The platform releases cash and reduces its liability to the customer.

```
Debit:  Wallet:Alice (Liability)  -$50     ← Platform owes Alice less
Credit: Cash (Asset)        -$50     ← Platform's cash decreases
```

| Account | Type | Direction | Amount | Effect |
|---|---|---|---|---|
| Wallet:Alice | Liability | **Debit** | 5000 | Liability decreases (opposite of normal balance) |
| Cash | Asset | **Credit** | 5000 | Asset decreases (opposite of normal balance) |

**Verify equation:** Assets (-50) = Liabilities (-50) + Equity (0) ✅

---

### 5.3 Transfer — "User A sends money to User B"

The platform's total liability doesn't change — it just shifts from one user to another. No cash moves.

```
Debit:  Wallet:Alice (Liability)  -$25     ← Alice's balance decreases
Credit: Wallet:Bob (Liability)    +$25     ← Bob's balance increases
```

| Account | Type | Direction | Amount | Effect |
|---|---|---|---|---|
| Wallet:Alice | Liability | **Debit** | 2500 | Liability to Alice decreases |
| Wallet:Bob | Liability | **Credit** | 2500 | Liability to Bob increases |

**Verify equation:** Assets (0) = Liabilities (-25 + 25 = 0) + Equity (0) ✅

---

## 6. Balance Calculation Formula

Since balance is **derived** from entries (never stored):

```sql
-- For Asset and Expense accounts (normal balance = debit):
balance = SUM(debit amounts) - SUM(credit amounts)

-- For Liability, Equity, and Revenue accounts (normal balance = credit):
balance = SUM(credit amounts) - SUM(debit amounts)
```

### In SQL:
```sql
SELECT
    CASE
        WHEN a.type IN ('asset', 'expense') THEN
            COALESCE(SUM(CASE WHEN e.direction = 'debit' THEN e.amount ELSE 0 END), 0) -
            COALESCE(SUM(CASE WHEN e.direction = 'credit' THEN e.amount ELSE 0 END), 0)
        ELSE
            COALESCE(SUM(CASE WHEN e.direction = 'credit' THEN e.amount ELSE 0 END), 0) -
            COALESCE(SUM(CASE WHEN e.direction = 'debit' THEN e.amount ELSE 0 END), 0)
    END AS balance
FROM accounts a
LEFT JOIN entries e ON e.account_id = a.id
WHERE a.id = $1
GROUP BY a.type;
```

### Historical Balance:
Add a `WHERE e.created_at <= $timestamp` filter to the entries join:
```sql
LEFT JOIN entries e ON e.account_id = a.id AND e.created_at <= $2
```

---

## 7. Immutability & Reversals

Ledger entries are **immutable** — they can never be updated or deleted. If a mistake is made, the correction is a **reversal transaction**: a new transaction with the debits and credits swapped.

### Example: Reversing a $100 deposit

**Original transaction:**
| Entry | Account | Direction | Amount |
|---|---|---|---|
| 1 | Cash | Debit | 10000 |
| 2 | Wallet:Alice | Credit | 10000 |

**Reversal transaction:**
| Entry | Account | Direction | Amount |
|---|---|---|---|
| 3 | Cash | Credit | 10000 |
| 4 | Wallet:Alice | Debit | 10000 |

After both transactions, the net effect on all accounts is **zero**. The original entries remain in the ledger as a permanent audit trail.

> [!TIP]
> This is how real accounting systems work. Tax authorities and auditors require a complete, unaltered trail of every financial event.
