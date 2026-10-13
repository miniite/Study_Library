---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a rollback-and-retry wrapper in PL/SQL that safely re-runs the failed transaction on ORA-00060?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Retry Strategy](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Retry_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of deadlocks in PL/SQL, we need to design a robust rollback-and-retry wrapper around transactions that can handle the ORA-00060 error, which indicates a deadlock. The core idea is to identify the failing transaction, roll it back, and then retry the entire block of code until either no more deadlocks occur or a maximum number of retries are reached.

