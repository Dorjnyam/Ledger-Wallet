# v2+ Backlog — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Roadmap](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/roadmap.md), [Project Charter](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/00-project-charter.md)

---

## 1. Objective / Purpose

This document captures features, improvements, and ideas that are **out of scope for v1** but worth building after the assignment is delivered. These are ordered by potential impact, not priority.

---

## 2. Feature Backlog

| ID | Feature | Category | Notes |
|---|---|---|---|
| V2-001 | **Multi-currency support** | Ledger | Allow transactions across currencies with exchange rate tracking |
| V2-002 | **gRPC API** | API | Add gRPC alongside REST for inter-service communication |
| V2-003 | **Event sourcing / audit log** | Ledger | Emit domain events on every transaction for downstream consumers |
| V2-004 | **Admin dashboard UI** | Frontend | Simple web UI to view accounts, balances, and transaction history |
| V2-005 | **Batch transaction posting** | Ledger | Accept multiple transactions in a single API call |
| V2-006 | **Account hierarchies** | Ledger | Chart of Accounts with parent-child relationships |
| V2-007 | **Soft account closure** | Accounts | Mark accounts as closed; reject new entries against them |
| V2-008 | **Pagination on transaction history** | API | Cursor-based pagination for large account histories |
| V2-009 | **Prometheus metrics** | Observability | Track transaction throughput, latency, error rates |
| V2-010 | **Rate limiting** | API | Protect against abuse on wallet endpoints |
| V2-011 | **Webhook notifications** | Integration | Notify external systems on deposits / transfers |
| V2-012 | **Scheduled / recurring transactions** | Wallet | Cron-based automatic transfers (e.g., monthly fees) |

---

## 3. Technical Debt to Address

| Item | Description |
|---|---|
| Connection pooling tuning | Profile `pgxpool` settings under load |
| Structured logging | Replace `log.Printf` with `slog` structured logger |
| Request tracing | Add OpenTelemetry trace IDs to all API requests |
| CI/CD pipeline | GitHub Actions for `go test`, `go vet`, `golangci-lint` |

> [!NOTE]
> This backlog is a living document. Add items as they surface during development. Don't let v2 ideas creep into v1 — log them here and move on.
