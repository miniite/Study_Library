---
type: "practice"
difficulty: "Intermediate"
---

# In a data pipeline, why is ROW_NUMBER() commonly used to deduplicate rows from a source table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deterministic Deduplication](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ROW_NUMBER() is a powerful window function often employed in SQL for deduplicating rows by assigning a unique sequential integer to rows within a partition of a result set, which helps in filtering out duplicates effectively. In data pipelines, this function simplifies the process of identifying and removing duplicate records without needing complex subqueries or joins.

To understand why ROW_NUMBER() is commonly used, we first need to clarify what it does and how it compares with other techniques like DISTINCT and GROUP BY. ROW_NUMBER() allows us to rank rows within a window based on some criteria, which can then be used to filter out duplicates more efficiently than using DISTINCT alone.

Let's start by breaking down the question from first principles:

Understanding the Problem: The primary goal is to identify and remove duplicate rows in a dataset.
Key Requirements: We need a mechanism that allows us to rank or order rows, so we can filter out those with the same values but different row identifiers.

Now, let's see how ROW_NUMBER() accomplishes this:

Step-by-Step Solution

Partitioning and Ordering:

PARTITION BY Clause: We use PARTITION BY to divide our data into partitions based on certain columns that we want to check for duplicates.
ORDER BY Clause: Within each partition, we sort the rows using an ORDER BY clause. This ensures that all identical values are grouped together.

Assigning ROW Numbers:

The ROW_NUMBER() function is then applied within this context. It assigns a unique number to each row within its partition and ordering.
By default, ROW_NUMBER() starts with 1 for the first row in each partition.

Filtering Duplicates:

To deduplicate rows, we can use a WHERE clause to filter out any row that has more than one occurrence (i.e., ROW_NUMBER() > 1).

#### Example

Let's consider a table named customer_orders, where we want to remove duplicate entries based on the combination of customer_id and order_date.

SQL
SELECT *
FROM (
    SELECT
        customer_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id, order_date ORDER BY order_id) as row_num
    FROM customer_orders
) AS subquery
WHERE row_num = 1;

In this example:

We partition the data by customer_id and order_date.
Within each partition, we sort by order_id, which can be a surrogate key or any other unique identifier.
We then select only those rows where row_num is 1.

#### Closing Summary

ROW_NUMBER() is commonly used in data pipelines to deduplicate rows because it allows for precise control over the ordering and ranking of records. This makes it easier to filter out duplicates without complex subqueries or joins, leading to cleaner and more maintainable code.

#### Common Mistakes
and Follow-Ups
Misunderstanding PARTITION BY: Sometimes, developers might forget to partition correctly, which can lead to incorrect deduplication.
Using ROW_NUMBER() Incorrectly: Forgetting to use the WHERE clause to filter out duplicate rows is another common mistake.

#### Performance Considerations
While ROW_NUMBER() is powerful, it might not be the most efficient solution for very large datasets. In such cases, other techniques like using a temporary table or a CTE (Common Table Expression) might offer better performance.

**Comparison:**

| customer_id | order_date       | order_id |
|-------------|------------------|----------|
| 101         | 2023-01-01       | 1        |
| 101         | 2023-01-01       | 2        |
| 102         | 2023-01-01       | 3        |
| 103         | 2023-01-02       | 4        |

**Code:**

```sql
SELECT *
FROM (
    SELECT
        customer_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id, order_date ORDER BY order_id) as row_num
    FROM customer_orders
) AS subquery
WHERE row_num = 1;
```

**Example input / output:**

```text
Input:
{
    "customer_orders": [
        {"customer_id": 101, "order_date": "2023-01-01", "order_id": 1},
        {"customer_id": 101, "order_date": "2023-01-01", "order_id": 2},
        {"customer_id": 102, "order_date": "2023-01-01", "order_id": 3},
        {"customer_id": 103, "order_date": "2023-01-02", "order_id": 4}
    ]
}

Output:
[
    {"customer_id": 101, "order_date": "2023-01-01", "order_id": 1},
    {"customer_id": 102, "order_date": "2023-01-01", "order_id": 3},
    {"customer_id": 103, "order_date": "2023-01-02", "order_id": 4}
]
```

**Data:**

```json
{
    "customer_orders": [
        {"customer_id": 101, "order_date": "2023-01-01", "order_id": 1},
        {"customer_id": 101, "order_date": "2023-01-01", "order_id": 2},
        {"customer_id": 102, "order_date": "2023-01-01", "order_id": 3},
        {"customer_id": 103, "order_date": "2023-01-02", "order_id": 4}
    ]
}
```

**Flow:**

1. Start
2. Partition by columns
3. Order rows within partitions
4. Assign ROW_NUMBER()
5. Filter rows with row_num = 1
6. End

