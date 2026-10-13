---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to compute the number of rows loaded per source file so you can spot a partial or empty file?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conditional Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write an SQL query that computes the number of rows loaded per source file, we need to first understand the structure of our data and how files are loaded into the database. This involves identifying a unique identifier for each file and counting the rows associated with it.

#### Triaging the Question

The core of the question is about writing an SQL query to count the number of rows from each file. The key idea is to use a GROUP BY clause to aggregate data by file, which will give us a summary of how many rows were loaded per file. We need to assume that there's a column in our table that uniquely identifies the source file.

#### First-Principles Solution
Identify the File Identifier Column: Assume we have a table loaded_data with columns like file_id, data_column1, data_column2, etc., where file_id uniquely identifies each file.
Use GROUP BY Clause: Group the rows by file_id and count them.

#### Example
Query
SQL
SELECT
    file_id,
    COUNT(*) AS row_count
FROM
    loaded_data
GROUP BY
    file_id;

#### Closing Summary
+ Common Mistakes/Follow-Ups

This query will give us a summary of the number of rows from each source file. However, there are some common pitfalls to watch out for:

Ensure that file_id is indeed unique and not null.
If the table has multiple files but they were loaded in batches (e.g., partitioned by date), you might need additional columns like load_date.

#### Cons
ider adding conditions to filter specific files if necessary.

#### Common Mistakes
Using an Inappropriate Column: Using a column that is not unique or does not uniquely identify the file.
Incorrect Aggregation Function: Using a wrong aggregation function (e.g., using SUM instead of COUNT).
Ignoring Null Values: Not handling null values properly in file_id.

**Comparison:**

| file_id | data_column1 | data_column2 |
|---------|--------------|--------------|
| 1       | value1       | value2       |
| 1       | value3       | value4       |
| 2       | value5       | value6       |
| 2       | value7       | value8       |
| file_id | row_count |
|---------|-----------|
| 1       | 2         |
| 2       | 2         |

**Code:**

```sql
SELECT
    file_id,
    COUNT(*) AS row_count
FROM
    loaded_data
GROUP BY
    file_id;
```

**Example input / output:**

```text
Input:
file_id | data_column1 | data_column2
1       | value1       | value2
1       | value3       | value4
2       | value5       | value6
2       | value7       | value8

Output:
file_id | row_count
1       | 2
2       | 2
```

**Data:**

```
loaded_data: A table with columns file_id, data_column1, and data_column2.
JSON
Copy
{
  "columns": ["file_id", "data_column1", "data_column2"],
  "rows": [
    {"file_id": 1, "data_column1": "value1", "data_column2": "value2"},
    {"file_id": 1, "data_column1": "value3", "data_column2": "value4"},
    {"file_id": 2, "data_column1": "value5", "data_column2": "value6"},
    {"file_id": 2, "data_column1": "value7", "data_column2": "value8"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a unique file identifier?
5. Use GROUP BY clause
6. Choose an appropriate columnand ensure uniqueness
7. Write the SQL query usingCOUNT(*)
8. Modify the schema to includea unique identifier
9. Execute the query and reviewresults
10. End

