# Bug Tracker — General Ledger & Wallet Service

**Status:** Living Document  
**Author:** Devshil  
**Last Updated:** 2026-08-20  
**Related Docs:** [Test Plan](file:///c:/Users/dell.000/Desktop/Ledger_wallet/docs/phase-6-testing/test-plan.md)

---

## 1. Objective / Purpose

Track **known bugs**, **issues encountered**, and their **resolutions** during development. This is a living document — update it as bugs are found and fixed.

---

## 2. Bug Log

| ID | Date | Severity | Description | Status | Resolution |
|---|---|---|---|---|---|
| BUG-001 | — | — | *(Log bugs here as they are discovered)* | — | — |

### Severity Levels

| Level | Meaning |
|---|---|
| **Critical** | Breaks a ledger invariant (debits ≠ credits, negative balance, data loss) |
| **High** | Functional failure (endpoint returns wrong data, test fails) |
| **Medium** | Non-blocking issue (poor error message, missing validation edge case) |
| **Low** | Cosmetic or minor (log formatting, naming inconsistency) |

---

## 3. Template for New Bugs

When adding a bug, include:

```markdown
| BUG-XXX | YYYY-MM-DD | Severity | Description of what went wrong | Open/Fixed | How it was fixed (or "investigating") |
```

> [!TIP]
> Don't just log bugs — log **how you found them** and **how you fixed them**. This is valuable during the code review walkthrough.
