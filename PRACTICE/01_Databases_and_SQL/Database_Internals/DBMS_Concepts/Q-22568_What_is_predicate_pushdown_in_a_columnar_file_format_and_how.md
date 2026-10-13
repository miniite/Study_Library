---
type: "practice"
difficulty: "Deep"
---

# What is predicate pushdown in a columnar file format and how does it reduce the amount of data scanned?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Index Access Paths](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Predicate pushdown is a technique used in databases, particularly with columnar storage formats like Parquet or ORC, to optimize query performance by filtering data at the storage level before transferring it to memory or processing. In essence, it pushes down predicates (conditions) from the execution plan of a query directly into the storage layer, reducing the amount of data that needs to be read and processed.

To understand predicate pushdown deeply, let's break it down step-by-step:

Understanding Columnar Storage: Columnar storage formats store data in columns rather than rows. This means all values for one column are grouped together, making it efficient to scan large datasets but potentially less so when dealing with small or specific ranges of data.

Query Execution Plan: When a query is executed, the database generates an execution plan that includes multiple steps: fetching data from storage, filtering, sorting, joining, and aggregating. Predicate pushdown occurs within this plan by moving predicate conditions directly into the storage layer.

#### Triage
and Assumptions:

We assume the query involves a filter condition (predicate) on one or more columns.
The storage format supports predicate pushdown optimizations.
The database management system (DBMS) can restructure queries to leverage these optimizations effectively.

#### First Principles Solution

Step 1: Identify Filter Conditions: When parsing the query, identify any predicates that can be applied directly at the storage level. For example, WHERE age > 30 AND city = 'New York'.
Step 2: Pushdown to Storage Layer: The DBMS modifies the execution plan by pushing these conditions into the storage layer, which then reads only the relevant columns and rows that match the predicate.
Step 3: Reduce Data Scanned: By filtering data at the storage level, a significant amount of data is skipped during transfer, reducing I/O operations and memory usage.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Predicate pushdown significantly reduces the amount of data scanned by moving filters to where the data resides.

#### Common mistakes
include not fully leveraging storage optimizations or incorrectly identifying predicates that can be pushed down.

#### Follow-ups
might involve discussing performance implications, trade-offs with other optimization techniques like indexing, and implementation details in specific DBMS systems.

**Comparison:**

| Step | Action |
|------|--------|
| 1    | Parse query and identify predicates.        |
| 2    | Modify execution plan for predicate pushdown.|
| 3    | Filter data at storage layer.               |
| 4    | Transfer filtered data to memory or processing stage. |

**Code:**

```python
# Example SQL Query for Predicate Pushdown
query = "SELECT * FROM users WHERE age > 30 AND city = 'New York';"
```

**Example input / output:**

```text
Input Query:
SELECT * FROM users WHERE age > 30 AND city = 'New York';

Expected Output:
user_id | name  | age | city
---------------------------------
       3 | Charlie |  35 | New York

Execution Plan with Predicate Pushdown:
1. Storage layer filters rows where age > 30 and city = 'New York'.
2. Only row (3, "Charlie", 35, "New York") is transferred to memory.
```

**Data:**

```json
{
  "table": {
    "columns": ["user_id", "name", "age", "city"],
    "data": [
      {"user_id": 1, "name": "Alice", "age": 25, "city": "New York"},
      {"user_id": 2, "name": "Bob", "age": 40, "city": "Los Angeles"},
      {"user_id": 3, "name": "Charlie", "age": 35, "city": "New York"}
    ]
  }
}
```

**Flow:**

1. No Predicate Pushdown
2. Predicate Identified
3. Parse Query
4. Identify Predicates
5. Maintain Original Plan
6. Execute Full Scan
7. Pushdown to Storage Layer
8. Filter Data at Storage Level
9. Transfer Filtered Data toMemory or Processing Stage

