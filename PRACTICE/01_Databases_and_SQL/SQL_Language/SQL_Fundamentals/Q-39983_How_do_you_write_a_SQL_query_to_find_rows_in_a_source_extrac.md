---
type: "practice"
difficulty: "Basic"
---

# How do you write a SQL query to find rows in a source extract that violate a NOT NULL business rule before loading them into the warehouse?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ NULL Handling](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write an SQL query that identifies any rows in our source extract which have null values in columns that are marked as NOT NULL by our business rules. This ensures data integrity before loading into the warehouse.

#### Triaging the Question

The question is asking for a SQL query to identify and flag rows with null values in specific columns. The interviewer might be probing your understanding of SQL syntax, how to handle NULL values, and your ability to apply logical conditions effectively.

#### Assumptions
We have a table called source_extract containing the data we want to check.
There are certain columns that must not contain null values according to our business rules.
The columns violating the NOT NULL rule will be identified by their names, which you provide as input or are known in advance.
Step-by-Step Solution

Identify the Columns: First, we need to know the column(s) where a NOT NULL violation should occur. Let's assume these are column1, column2, and column3.

#### Cons
truct the Query:

We will use the WHERE clause to filter out rows that violate the NOT NULL rule.
If any of the specified columns contain null values, those rows will be selected.

Here’s how you can write this query:

SQL
SELECT *
FROM source_extract
WHERE column1 IS NULL OR column2 IS NULL OR column3 IS NULL;

This query will return all rows from source_extract where at least one of the specified columns (column1, column2, or column3) is null. This ensures that any row violating a NOT NULL rule stands out.

#### Closing Summary
and Common Mistakes

To wrap up, we've identified the key step of using the IS NULL operator to detect null values in specific columns. It’s essential to ensure you understand which columns are supposed to have non-null values before writing this query.

#### Common Mistakes
Incorrect Column Names: Double-check that the column names match exactly with those used in your database schema.
Improper Use of Logical Operators: Ensure you use OR correctly if multiple columns might be null, or use AND if all columns must not be null for a row to violate the rule.

#### Example
Input-Output

For instance, if our source extract has the following data:

id	column1	column2	column3
1	NULL	'value'	'value'
2	'value'	NULL	'value'
3	'value'	'value'	NULL

Running the query would return:

id	column1	column2	column3
1	NULL	'value'	'value'
2	'value'	NULL	'value'
3	'value'	'value'	NULL

This output shows us which rows violate the NOT NULL rule.

**Code:**

```sql
SELECT *
FROM source_extract
WHERE column1 IS NULL OR column2 IS NULL OR column3 IS NULL;
```

**Example input / output:**

```text
-- Input:
SELECT *
FROM source_extract
WHERE column1 IS NULL OR column2 IS NULL OR column3 IS NULL;

-- Output:
+----+---------+---------+---------+
| id | column1 | column2 | column3 |
+----+---------+---------+---------+
| 1  | NULL    | 'value' | 'value' |
| 2  | 'value' | NULL    | 'value' |
| 3  | 'value' | 'value' | NULL    |
+----+---------+---------+---------+
```

**Data:**

| id | column1 | column2 | column3 |
|----|---------|---------|---------|
| 1  | NULL    | 'value' | 'value' |
| 2  | 'value' | NULL    | 'value' |
| 3  | 'value' | 'value' | NULL    |

