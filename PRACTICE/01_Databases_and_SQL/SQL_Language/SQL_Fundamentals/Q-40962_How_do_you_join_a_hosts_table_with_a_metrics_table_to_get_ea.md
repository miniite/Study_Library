---
type: "practice"
difficulty: "Basic"
---

# How do you join a hosts table with a metrics table to get each metric row alongside the host's data-center location?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Completeness Metrics](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Completeness_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To join a hosts table with a metrics table and get each metric row alongside the host's data-center location, we need to use an SQL JOIN operation. The key idea is to match records between the two tables based on a common column (such as a host ID) that uniquely identifies each host in both tables.

#### Triaging the Question

The question asks for how to join two tables: hosts and metrics. Specifically, it requires us to include information from the hosts table (which contains data-center location details) alongside the corresponding metric rows in the metrics table. The core of this problem is understanding SQL JOINs and knowing which type to use based on our requirements.

#### First-Principles Solution
Identify Common Columns: We need a common column that links entries between the two tables, such as a host ID.
Choose the Appropriate Join Type: Typically, an INNER JOIN is sufficient for this task if we only want records where both tables have matching values in their respective join columns.
Include Relevant Columns: We need to select columns from both tables that are relevant to our output.
SQL Query

Here’s a step-by-step breakdown of the query:

SQL
SELECT
    h.id AS host_id,
    h.data_center_location AS data_center,
    m.metric_name,
    m.value,
    m.timestamp
FROM
    hosts h
INNER JOIN
    metrics m ON h.id = m.host_id;

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake 1: Using LEFT JOIN or RIGHT JOIN instead of INNER JOIN when an exact match is required.
Mistake 2: Forgetting to select relevant columns from both tables.

#### Follow-Up
Discuss how this query performs in different database systems and whether there are any indexing considerations for the join column.

**Code:**

```sql
SELECT
    h.id AS host_id,
    h.data_center_location AS data_center,
    m.metric_name,
    m.value,
    m.timestamp
FROM
    hosts h
INNER JOIN
    metrics m ON h.id = m.host_id;
```

