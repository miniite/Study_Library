---
type: "practice"
difficulty: "Deep"
---

# How would you compute a sessionized funnel (view then add-to-cart then purchase in order within a session) in warehouse SQL, and why do naive self-joins on event type explode the row count?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Join Fan-out](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Fan-out.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a sessionized funnel in Warehouse SQL, specifically focusing on the sequence of events such as "view", "add-to-cart", and "purchase" within the same session, we need to identify sessions first and then filter events based on their order. Naive self-joins can lead to exponential growth in row counts due to the nature of joining multiple tables or partitions. Let's break this down from first principles.

