---
type: "practice"
difficulty: "Deep"
---

# How would you use dynamic tables to replace a hand-built Streams-and-Tasks pipeline, and when is that a bad idea?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Snowflake Loading and Pipelines](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Loading_and_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Dynamic tables in Snowflake offer a powerful feature set to process data on-the-fly without explicitly setting up streams and tasks. This can significantly simplify ETL processes by abstracting away many of the complexities involved in managing separate streams and tasks. However, it is not always the best choice, and certain scenarios may indicate that a hand-built Streams-and-Tasks pipeline is more appropriate.

To understand when to use dynamic tables versus Streams-and-Tasks, let's first break down what each approach entails:

#### Triage
Understanding Streams and Tasks vs Dynamic Tables
Streams
Streams are used to automatically capture changes in a table or view.
They trigger tasks (e.g., stored procedures) for processing these changes.
Tasks
Tasks can be scheduled to run at specific intervals or based on stream events.
They provide the actual logic to process and transform data.
Dynamic Tables
Dynamic tables automatically capture inserts, updates, and deletes in a table.
They expose an updatable view of these changes, allowing you to query them directly without needing explicit tasks for processing.

#### First-Principles Solution

To replace a hand-built Streams-and-Tasks pipeline with dynamic tables:

Identify the Source Table:

Define the source table that will have dynamic data changes.

Enable Dynamic Updates:

Use ALTER TABLE to enable dynamic updates on the source table.

Create a Queryable View:

Create a view that queries the dynamic table, effectively abstracting away the need for explicit tasks.

Process Data Directly in SnowSQL or APIs:

Write queries directly against this view to process and transform data as needed.

Monitor Performance and Integrity:

Regularly monitor performance and ensure data integrity is maintained without requiring separate task execution.
Detailed Steps

Enable Dynamic Updates on the Source Table:

SQL
ALTER TABLE source_table ENABLE ROW ACCESS POLICY (ROW_NUMBER() OVER () <= 100);

Create a Queryable View:

SQL
CREATE OR REPLACE VIEW dynamic_view AS
SELECT * FROM source_table;

Query and Process Data:

SQL
-- Example query to process data directly from the view
SELECT col1, col2, MAX(col3) OVER (PARTITION BY col1) as max_col3
FROM dynamic_view
WHERE col4 > 'some_value';

#### Closing Summary

Using dynamic tables can simplify ETL processes by abstracting away many of the complexities involved in managing separate streams and tasks. However, it's essential to consider factors such as performance, data integrity, and complexity before making this change.

#### Common Mistakes/Follow-Ups
Over-reliance on Dynamic Tables: Not all data processing scenarios benefit from dynamic tables; complex transformations may require explicit task execution.
Performance Issues: Large volumes of changes can cause performance bottlenecks in the view layer.
Data Integrity Concerns: Ensure that no external systems are relying on the stream and task architecture, as this could lead to integrity issues.

In summary, while dynamic tables offer significant convenience, a hand-built Streams-and-Tasks pipeline may still be necessary for complex or high-performance requirements.

**Code:**

```sql
ALTER TABLE source_table ENABLE ROW ACCESS POLICY (ROW_NUMBER() OVER () <= 100);
CREATE OR REPLACE VIEW dynamic_view AS
SELECT * FROM source_table;
```

