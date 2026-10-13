---
type: "practice"
difficulty: "Deep"
---

# How would you build a point-in-time-correct join so a report shows facts against the dimension as it looked at the event date, and how do you index it to stay fast?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Transaction vs Snapshot Facts](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a point-in-time-correct join for a data model that accurately reflects dimensions as they were on an event date, we need to create a snapshot or historical version of our dimension tables and ensure our fact table is properly linked to these snapshots. The core idea here is to maintain the consistency of the data by joining at the event's timestamp rather than the current state. This approach ensures that reports are based on accurate historical contexts.

#### Triaging the Question

The key aspects we need to address include:

Understanding the Requirement: Ensuring that each fact in our report is linked to the dimension as it existed when the event occurred, not at some arbitrary point in time.
Database Schema Design: Creating a star or snowflake schema with appropriate snapshot tables.
Join Strategy: Implementing a join that captures the state of the dimensions based on the event date.
Indexing and Performance Optimization: Ensuring fast performance by optimizing indexes to quickly retrieve relevant data.

#### First-Principles Solution
Dimension Table Snapshots: We will maintain historical snapshots of our dimension tables, creating new records each time a change is made.
Fact Table Schema: Each fact record in the fact table should reference a specific event date and its corresponding snapshot IDs from the dimension tables.
Join Mechanism: When generating reports, we join the facts with their respective dimension snapshots based on the event date.
Detailed Steps

Designing Snapshots:

For each dimension table, create a corresponding "snapshot" or "history" table that logs changes over time.
Each snapshot record will capture the state of the dimension at a specific point in time (e.g., event_date).

Updating Fact Table:

When a fact is inserted, it should reference the appropriate snapshot IDs from each dimension table. This can be stored as foreign keys or additional columns.

Querying with Point-in-Time Correctness:

To generate reports, we query the facts and join them to their respective dimension snapshots using the event_date in the fact record.
The SQL query might look something like:
SQL
SELECT f.*, d1.*, d2.*
FROM facts f
JOIN dim_snapshot d1 ON (f.dim_id_1 = d1.id AND f.event_date = d1.event_date)
JOIN dim_snapshot d2 ON (f.dim_id_2 = d2.id AND f.event_date = d2.event_date);

Indexing Strategy:

Create indexes on the event_date column in both fact and dimension snapshot tables.

#### Cons
ider compound indexes that include both event_date and relevant foreign keys.

#### Closing Summary

By implementing this approach, we ensure that reports are accurate reflections of historical data points at specific times. Proper indexing ensures fast query performance even with large datasets. Common pitfalls include not maintaining consistent snapshot IDs or missing proper indexing on critical columns.

#### Common Mistakes
and Follow-Ups
Forgetting to Update Snapshots: Ensure all changes to dimensions are reflected in the snapshots.
Indexing Overhead: Carefully manage index sizes to avoid performance bottlenecks.
Query Optimization: Regularly review query performance and adjust indexing strategies as needed.

**Comparison:**

```
table "Dim_Snapshot" {
  "| id | event_date | other_columns... |"
  "| ---| ---------- | --------------- |"
  "| 101 | 2023-06-01 | ...             |"
}
table "Facts" {
  "| fact_id | dim_id_1 | dim_id_2 | event_date |"
  "| -------:| --------:| --------:| ----------:|"
  "|       1 |        101 |        203 | 2023-06-01 |"
}
```

**Code:**

```sql
-- Example of creating snapshot tables
CREATE TABLE dim_snapshot (
  id INT PRIMARY KEY,
  event_date DATE NOT NULL,
  other_columns ...
);

-- Inserting data into snapshots
INSERT INTO dim_snapshot (id, event_date, ...)
VALUES (?, ?, ...);
```

**Example input / output:**

```

Input:

SQL
Copy
INSERT INTO dim_snapshot (id, event_date) VALUES (101, '2023-06-01');
Output:
PLAINTEXT
Copy
Snapshot record created for dimension ID 101 on 2023-06-01.

Input:

SQL
Copy
INSERT INTO facts (fact_id, dim_id_1, dim_id_2, event_date) VALUES (?, ?, ?, '2023-06-01');
Output:
PLAINTEXT
Copy
Fact record created with snapshot IDs and event date.
```

**Data:**

```text
- `dim_snapshot`: Contains historical records of dimension changes.
- `facts`: Stores facts with references to snapshot IDs and event dates.
```

**Flow:**

1. Yes
2. No
3. Fact Table
4. Join on Event Date?
5. Retrieve Snapshot
6. Fall back to Current State
7. Generate Report with JoinedData
8. Generate Report with CurrentDimension Data

