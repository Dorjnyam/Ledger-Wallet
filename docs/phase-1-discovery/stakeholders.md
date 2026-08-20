# Stakeholders — General Ledger & Wallet Service

**Status:** Draft  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Vision](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-1-discovery/vision.md), [Project Charter](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/00-project-charter.md)

---

## 1. Objective / Purpose

Identify all stakeholders and users of the Ledger & Wallet Service — who interacts with the system, and what they need from it.

---

## 2. Stakeholders

| Role | Description | Needs |
|---|---|---|
| **Reviewer / Evaluator** | The person who reads the code, runs the demo, and asks questions | Clean code, passing tests, clear docs, defensible design decisions |
| **Developer (You)** | The person building and maintaining the system | Clear requirements, good documentation, testable code |
| **API Consumer** | Any client application calling the REST API (e.g., a mobile app, admin dashboard, or another microservice) | Predictable endpoints, clear error codes, idempotent operations |
| **Platform Operator** | Anyone deploying and monitoring the service in production | Docker support, environment variable config, health checks |

---

## 3. User Personas

### Persona 1: The Evaluator
- **Goal:** Verify that the candidate understands double-entry bookkeeping and can implement it correctly in Go
- **Key Concerns:** Correctness of invariants, code quality, concurrency safety, ability to explain decisions
- **Interaction:** Reads README → runs `docker compose up` → hits API with `curl` → reads code → asks questions

### Persona 2: The API Consumer
- **Goal:** Manage wallets (deposit, withdraw, transfer) via a reliable API
- **Key Concerns:** API must be idempotent (safe to retry), must return clear error messages, must never allow overdraft
- **Interaction:** Sends JSON requests → receives JSON responses with appropriate HTTP status codes

### Persona 3: The Future Developer
- **Goal:** Extend the system with new features (multi-currency, audit log, etc.)
- **Key Concerns:** Clean architecture, separation of concerns, documented design decisions
- **Interaction:** Reads docs → understands layered architecture → adds features without breaking invariants

---

## 4. Stakeholder Communication

| Stakeholder | Communication Channel | Frequency |
|---|---|---|
| Evaluator | README, docs folder, live demo, code review | End of 5-day period |
| API Consumer | API spec, error messages, HTTP status codes | Every API call |
| Future Developer | Docs folder, code comments, architecture diagrams | Ongoing |
