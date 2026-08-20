# API Specification — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Architecture](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-4-design/architecture.md), [Functional Requirements](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-2-requirements/functional-requirements.md)

---

## 1. Objective / Purpose

Define every REST API endpoint — method, path, request body, response body, and status codes. This is the contract between the service and any API consumer.

---

## 2. Base URL

```
http://localhost:8080/v1
```

---

## 3. Endpoints

### 3.1 Accounts

#### `POST /v1/accounts` — Create Account

**Request:**
```json
{
  "name": "Cash",
  "type": "asset",
  "currency": "USD"
}
```

**Response (201 Created):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Cash",
  "type": "asset",
  "currency": "USD",
  "created_at": "2026-08-20T10:00:00Z"
}
```

**Error Responses:**
| Status | Condition |
|---|---|
| 400 | Missing required fields or invalid account type |

---

#### `GET /v1/accounts/:id` — Get Account

**Response (200 OK):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Cash",
  "type": "asset",
  "currency": "USD",
  "created_at": "2026-08-20T10:00:00Z"
}
```

**Error Responses:**
| Status | Condition |
|---|---|
| 404 | Account not found |

---

### 3.2 Transactions

#### `POST /v1/transactions` — Post Transaction

**Request:**
```json
{
  "idempotency_key": "txn-001-deposit-alice",
  "description": "Initial deposit for Alice",
  "entries": [
    {
      "account_id": "cash-account-uuid",
      "direction": "debit",
      "amount": 10000
    },
    {
      "account_id": "alice-wallet-uuid",
      "direction": "credit",
      "amount": 10000
    }
  ]
}
```

> [!NOTE]
> `amount` is in **minor units** (cents). `10000` = $100.00.

**Response (201 Created):**
```json
{
  "id": "transaction-uuid",
  "idempotency_key": "txn-001-deposit-alice",
  "description": "Initial deposit for Alice",
  "posted_at": "2026-08-20T10:00:00Z",
  "entries": [
    {
      "id": "entry-uuid-1",
      "account_id": "cash-account-uuid",
      "direction": "debit",
      "amount": 10000
    },
    {
      "id": "entry-uuid-2",
      "account_id": "alice-wallet-uuid",
      "direction": "credit",
      "amount": 10000
    }
  ]
}
```

**Error Responses:**
| Status | Condition |
|---|---|
| 400 | Missing fields, invalid account ID, fewer than 2 entries |
| 409 | Duplicate idempotency_key — returns existing transaction |
| 422 | Total debits ≠ total credits |

---

### 3.3 Balance

#### `GET /v1/accounts/:id/balance` — Current Balance

**Response (200 OK):**
```json
{
  "account_id": "alice-wallet-uuid",
  "balance": 10000,
  "currency": "USD"
}
```

---

#### `GET /v1/accounts/:id/balance?at=2026-08-15T12:00:00Z` — Historical Balance

**Response (200 OK):**
```json
{
  "account_id": "alice-wallet-uuid",
  "balance": 5000,
  "currency": "USD",
  "as_of": "2026-08-15T12:00:00Z"
}
```

**Error Responses:**
| Status | Condition |
|---|---|
| 400 | Invalid timestamp format |
| 404 | Account not found |

---

### 3.4 Transaction History

#### `GET /v1/accounts/:id/transactions` — Account Transaction History

**Response (200 OK):**
```json
{
  "account_id": "alice-wallet-uuid",
  "transactions": [
    {
      "id": "txn-uuid-1",
      "description": "Initial deposit",
      "posted_at": "2026-08-20T10:00:00Z",
      "entries": [
        {"account_id": "cash-uuid", "direction": "debit", "amount": 10000},
        {"account_id": "alice-wallet-uuid", "direction": "credit", "amount": 10000}
      ]
    }
  ]
}
```

---

### 3.5 Wallet Operations

#### `POST /v1/wallets/deposit` — Deposit

**Request:**
```json
{
  "wallet_id": "alice-wallet-uuid",
  "amount": 5000,
  "idempotency_key": "dep-001-alice",
  "description": "Salary deposit"
}
```

**Response (201 Created):** Returns the created transaction (same format as `POST /v1/transactions` response).

**Error Responses:**
| Status | Condition |
|---|---|
| 400 | Missing fields, invalid wallet ID |
| 409 | Duplicate idempotency_key — returns existing transaction |

---

#### `POST /v1/wallets/withdraw` — Withdraw

**Request:**
```json
{
  "wallet_id": "alice-wallet-uuid",
  "amount": 3000,
  "idempotency_key": "wd-001-alice",
  "description": "ATM withdrawal"
}
```

**Response (201 Created):** Returns the created transaction.

**Error Responses:**
| Status | Condition |
|---|---|
| 400 | Missing fields |
| 409 | Duplicate idempotency_key |
| 422 | Insufficient balance (would result in negative balance) |

---

#### `POST /v1/wallets/transfer` — Transfer

**Request:**
```json
{
  "source_wallet_id": "alice-wallet-uuid",
  "destination_wallet_id": "bob-wallet-uuid",
  "amount": 2000,
  "idempotency_key": "xfer-001-alice-bob",
  "description": "Payment to Bob"
}
```

**Response (201 Created):** Returns the created transaction.

**Error Responses:**
| Status | Condition |
|---|---|
| 400 | Missing fields, source = destination |
| 409 | Duplicate idempotency_key |
| 422 | Insufficient source balance |

---

## 4. Common Error Response Format

All errors return a consistent JSON structure:

```json
{
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Wallet balance of 3000 is insufficient for withdrawal of 5000"
  }
}
```

| Error Code | HTTP Status | Meaning |
|---|---|---|
| `INVALID_REQUEST` | 400 | Malformed request or missing required fields |
| `NOT_FOUND` | 404 | Account or transaction not found |
| `DUPLICATE_IDEMPOTENCY_KEY` | 409 | Idempotency key already used; existing result returned |
| `UNBALANCED_TRANSACTION` | 422 | Total debits ≠ total credits |
| `INSUFFICIENT_BALANCE` | 422 | Wallet balance too low for operation |
| `INTERNAL_ERROR` | 500 | Unexpected server error |
