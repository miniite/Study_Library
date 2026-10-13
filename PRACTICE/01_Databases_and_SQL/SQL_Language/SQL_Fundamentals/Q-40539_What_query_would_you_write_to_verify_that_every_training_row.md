---
type: "practice"
difficulty: "Basic"
---

# What query would you write to verify that every training row has a non-null label before feeding it to the model?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Validation Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To verify that every training row has a non-null label before feeding it to the model, we need to ensure our dataset is clean and consistent. This involves writing an SQL query to check if there are any rows in the training table where the label column is null.

The core idea here is straightforward: we want to count the number of rows with null labels or identify these rows directly. Let’s break this down step by step from first principles.

#### Triage

What the question is really asking and what the interviewer is probing:

The interviewer wants you to write an SQL query that checks for null values in a specific column (the label column).
They are testing your basic understanding of SQL, particularly in handling conditional queries.

#### Assumptions
include the existence of a training table with at least one label column.

#### First-Principles Solution
Identify the Table and Column: Assume our training data is stored in a table called train_data.
Check for Null Values: We need to select rows where the label is null.
Count or List the Rows: Depending on whether you just want a count or the actual rows, we can use a SELECT statement with appropriate conditions.

Here’s how we would write this query:

SQL
-- Counting the number of rows with null labels
SELECT COUNT(*)
FROM train_data
WHERE label IS NULL;

-- Listing all rows with null labels (optional)
SELECT *
FROM train_data
WHERE label IS NULL;

#### Closing Summary
and Common Mistakes/Follow-Ups

The above query will help us verify that no rows in the train_data table have a null label. If this count is non-zero, it indicates an issue with our data. The query can be extended to list the specific rows if necessary.

#### Common Mistakes

Forgetting to include the condition label IS NULL.
Using = instead of IS NULL, which would return no results as nulls are not equal to anything.
Not specifying the table name, leading to ambiguity in larger database schemas.

If you’re asked further questions, the interviewer might probe deeper into your understanding of data cleaning processes or how to integrate this check with other parts of your machine learning pipeline. For example:

How would you handle null labels if they are expected as part of the training process?
What additional steps would you take after identifying rows with null labels?

**Comparison:**

| Column Name | Data Type |
|-------------|-----------|
| id          | INT       |
| feature1    | FLOAT     |
| feature2    | VARCHAR   |
| label       | INT       |

**Code:**

```sql
-- Counting the number of rows with null labels
SELECT COUNT(*)
FROM train_data
WHERE label IS NULL;

-- Listing all rows with null labels (optional)
SELECT *
FROM train_data
WHERE label IS NULL;
```

**Data:**

```
train_data table schema:
SQL
Copy
CREATE TABLE train_data (
    id INT,
    feature1 FLOAT,
    feature2 VARCHAR(255),
    label INT
);
```

