---
type: "practice"
difficulty: "Deep"
---

# Explain the reliable outbox pattern in SQL so domain events publish exactly-once relative to the DB transaction, and where it can still duplicate or lose events.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Idempotency Key Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The reliable outbox pattern is a method used to ensure that domain events are published exactly-once relative to the database transaction in a distributed system. This pattern is particularly useful when integrating with external systems where message loss or duplication can lead to inconsistent states. By ensuring that messages are not sent until after the database transaction has committed, we can avoid scenarios where the event is lost if the transaction fails.

To understand how this works and its potential pitfalls, let's break it down from first principles.

