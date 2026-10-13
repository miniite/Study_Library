---
type: "practice"
difficulty: "Deep"
---

# For a Type 2 dimension, how would you write a JOIN that picks the dimension row that was effective at the fact's event date?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Joining Type 2 Dimensions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this SQL question, we need to understand what a Type 2 dimension is and how it handles changes over time. In a Type 2 dimension, each change creates a new record with an effective and inactive date range, allowing the historical data to be preserved. The challenge here is to join the fact table with the appropriate dimension row that was active on the event date of the fact.

To solve this, we need to write a SQL query that performs a JOIN between the fact table and the dimension table based on the effective dates. Here’s a step-by-step breakdown:

Identify the Effective Date in the Fact Table: Assume the fact table has columns fact_id, event_date, etc.
Identify the Effective Dates in the Dimension Table: The dimension table should have columns like dim_id, start_date, end_date, and possibly an effective_date if it’s explicitly stored.
Join on the Appropriate Condition: We need to find the dimension row that was active at the time of the fact event.

Let's write this query from first principles.

