---
type: "practice"
difficulty: "Deep"
---

# Explain how you would implement optimistic locking versus SELECT FOR UPDATE in a FastAPI endpoint that decrements inventory, and how each behaves under high write contention.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Timeouts and Resource Protection](<../../../../TOOLS/Python/FastAPI/Timeouts_and_Resource_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage inventory decrement operations efficiently while ensuring data consistency, we need to consider both optimistic locking and SELECT FOR UPDATE strategies in a FastAPI endpoint. Optimistic locking is a non-blocking approach where we assume that multiple concurrent transactions can proceed without interference until they attempt to commit their changes, at which point a conflict is detected. On the other hand, SELECT FOR UPDATE is a blocking mechanism where a transaction locks specific rows to prevent others from modifying them during its execution.

Optimistic locking can be implemented using versioning or timestamps in the database. However, it requires careful handling of retries and conflicts. SELECT FOR UPDATE, on the other hand, provides stronger guarantees but comes with higher lock contention and potential performance issues under high write contention scenarios.

Let's explore both approaches in detail for a FastAPI endpoint that decrements inventory.

