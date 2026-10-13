---
type: "practice"
difficulty: "Basic"
---

# Write a query to find the most frequent category value in a column so you can decide on encoding.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Aggregation Query Practice](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write an SQL query that finds the most frequently occurring category in a given column. This will help us understand which categories dominate our dataset and inform decisions about how to encode categorical data. The core idea is to use aggregation functions like COUNT and window functions or subqueries.

#### Triage

The question asks for an SQL query to find the most frequent category value, implying that we need to process a table with at least one column containing categories. We assume that the table name is categories_table and the column of interest is named category_column. The goal is to return the category with the highest frequency.

#### First-Principles Solution
Count Occurrences: Use the COUNT function to count how many times each category appears in the dataset.
Group by Category: Group the data by the category_column.
Find Maximum Count: Find the maximum count among all groups to identify the most frequent category.

Here is a step-by-step approach:

Initial Query Structure:

SQL
SELECT
    category_column,
    COUNT(*) as freq_count
FROM
    categories_table
GROUP BY
    category_column;

Identify the Most Frequent Category: To find the most frequent category, we can use a subquery or window function to identify the maximum frequency.

Full Solution

Using a subquery:

SQL
SELECT
    category_column,
    freq_count
FROM (
    SELECT
        category_column,
        COUNT(*) as freq_count
    FROM
        categories_table
    GROUP BY
        category_column
) AS freqs
WHERE
    freq_count = (SELECT MAX(freq_count) FROM freqs);

Alternatively, using a window function:

SQL
WITH freq_counts AS (
    SELECT
        category_column,
        COUNT(*) as freq_count
    FROM
        categories_table
    GROUP BY
        category_column
)
SELECT
    category_column
FROM
    freq_counts
WHERE
    freq_count = (SELECT MAX(freq_count) FROM freq_counts);

#### Closing Summary
+ Common Mistakes/Follow-Ups

The query above correctly identifies the most frequent category. However, it's worth noting that if there are ties for the maximum frequency, both approaches will return all tied categories. If you need to handle ties differently (e.g., returning multiple categories or a random choice), additional logic would be needed.

#### Common mistakes
include:

Not grouping by the category column.
Forgetting to use COUNT to count occurrences.
Not handling ties correctly if required.

For follow-ups, consider asking about performance optimization, data types, and potential edge cases such as empty tables or very large datasets.

**Comparison:**

| id  | category_column |
|-----|-----------------|
| 1   | apple           |
| 2   | banana          |
| 3   | apple           |
| 4   | orange          |
| 5   | apple           |
| 6   | banana          |

**Code:**

```sql
-- Example query with window function
WITH freq_counts AS (
    SELECT
        category_column,
        COUNT(*) as freq_count
    FROM
        categories_table
    GROUP BY
        category_column
)
SELECT
    category_column
FROM
    freq_counts
WHERE
    freq_count = (SELECT MAX(freq_count) FROM freq_counts);
```

**Example input / output:**

```text
-- Sample input and output

Input:
WITH freq_counts AS (
    SELECT
        category_column,
        COUNT(*) as freq_count
    FROM
        categories_table
    GROUP BY
        category_column
)
SELECT
    category_column
FROM
    freq_counts
WHERE
    freq_count = (SELECT MAX(freq_count) FROM freq_counts);

Output:
category_column
-----------------
apple
```

**Data:**

```sql
-- Sample data schema for testing
CREATE TABLE categories_table (
    id INT,
    category_column VARCHAR(100)
);

INSERT INTO categories_table (id, category_column) VALUES
(1, 'apple'),
(2, 'banana'),
(3, 'apple'),
(4, 'orange'),
(5, 'apple'),
(6, 'banana');
```

**Flow:**

1. Start
2. Count each category
3. Group by category and count
4. Find max count
5. Filter to get most frequentcategories
6. End

