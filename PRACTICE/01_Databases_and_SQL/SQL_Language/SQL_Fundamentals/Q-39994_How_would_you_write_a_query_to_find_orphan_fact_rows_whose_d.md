---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to find orphan fact rows whose dimension key does not exist in the dimension table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/RAG_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find orphan fact rows, we need to identify fact records where there is no corresponding entry in the dimension table based on a specific foreign key. This involves joining the fact table with the dimension table and then filtering out those rows that do not have a match.

#### Triage
Understanding the Question: The question asks for a SQL query to find "orphan" records in the fact table, which means finding records where there is no corresponding entry in the dimension table based on a foreign key.

#### Assumptions
We have two tables: fact_table and dimension_table.
The foreign key in fact_table is named dim_key.

#### Solution
Approach: We will perform an anti-join, which returns rows from the left table (fact_table) that do not have a match in the right table (dimension_table).

#### First Principles Solution
SQL
SELECT fact_table.*
FROM fact_table
LEFT JOIN dimension_table ON fact_table.dim_key = dimension_table.key
WHERE dimension_table.key IS NULL;

#### Closing Summary
+ Common Mistakes/Follow-Ups:

This query effectively filters out any fact_table records that do not have a corresponding entry in the dimension_table. The key here is to use a LEFT JOIN and check for null values on the right table’s foreign key. A common mistake might be forgetting to perform the anti-join correctly, leading to incorrect results.

#### Common Mistakes
Incorrect Join Type: Using an INNER JOIN instead of a LEFT JOIN will miss out on orphan records.
Misaligned Foreign Key Names: Ensuring that the column names used in the ON clause match exactly with those in both tables.
Handling NULL Values Incorrectly: Forgetting to check for NULL values after the join.
Further Considerations:
If performance is a concern, you might want to ensure indexes are placed on the foreign key columns.
Depending on your database system, there may be more efficient ways to achieve this query (e.g., using window functions in some databases).

**Code:**

```sql
SELECT fact_table.*
FROM fact_table
LEFT JOIN dimension_table ON fact_table.dim_key = dimension_table.key
WHERE dimension_table.key IS NULL;
```

