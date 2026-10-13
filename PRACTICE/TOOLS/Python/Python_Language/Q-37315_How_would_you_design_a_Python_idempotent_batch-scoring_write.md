---
type: "practice"
difficulty: "Deep"
---

# How would you design a Python idempotent batch-scoring write path so re-running a failed nightly job across billions of rows never double-writes to the warehouse, and make that pattern the org default?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotent Event Handlers](<../../../../TOOLS/Python/Python_Language/Idempotent_Event_Handlers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an idempotent batch-scoring write path in Python for handling large-scale data processing like a nightly job with billions of rows, we need to ensure that re-running the process does not result in double-writes to the warehouse. This involves implementing a mechanism that checks and updates records only when necessary. Here’s a step-by-step approach:

First, let's define what idempotence means: an operation can be applied multiple times without changing the result after the first application. For our use case, we want each row in the database to be updated exactly once even if the job is run multiple times.

To achieve this, we'll use a combination of transactional integrity and unique constraints. We will also implement a retry mechanism for failed jobs. The solution should handle both the database interactions and the parallel processing of large datasets effectively.

