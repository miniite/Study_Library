---
type: "practice"
difficulty: "Intermediate"
---

# How would you use the ROW_NUMBER window function to deduplicate records and keep only the latest version of each key?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Reject vs Quarantine Bad Rows](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the task of using the ROW_NUMBER window function to deduplicate records by keeping only the latest version of each key, we need to first understand that this is a common requirement in data preprocessing and cleaning scenarios where duplicate records with different versions or updates are present. The objective here is to identify the most recent record for each unique key and retain it while removing all other duplicates.

Essentially, the ROW_NUMBER function allows us to assign a unique row number within each partition of a result set. By partitioning by our key column and ordering by a timestamp or version number (assuming we have such metadata), we can effectively manage and filter out older records.

Let's break down the solution step-by-step:

Identify the Key Column: Determine which column(s) define the uniqueness of your records.
Add a Timestamp/Version Column: If not already present, include a timestamp or version number that indicates when each record was updated.
Apply ROW_NUMBER Window Function: Use ROW_NUMBER to assign unique row numbers within partitions defined by the key columns and ordered by the timestamp/version column.
Filter Rows: Select only the rows where ROW_NUMBER = 1, which will give us the latest version of each record.

Here’s a more detailed explanation:

#### Triaging the Question
The interviewer is probing your understanding of SQL window functions, specifically ROW_NUMBER.
You need to make assumptions about the data schema and any available metadata.
You'll be expected to demonstrate your ability to write complex SQL queries that handle partitioning and ordering.

#### First Principles Solution

Identify Key Columns: Suppose we have a table named orders with columns order_id, customer_id, product_name, quantity, order_date.

Add Timestamp Column (if necessary): If the data does not already include a timestamp, you might need to create one based on another date field or assume the most recent update time.

Window Function:

Use ROW_NUMBER() over partitioned data by key columns.
Order by the timestamp column in descending order to ensure newer records get row number 1.

Here’s an example query:

SQL
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        product_name,
        quantity,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id, product_name ORDER BY order_date DESC) as rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    product_name,
    quantity,
    order_date
FROM ranked_orders
WHERE rn = 1;

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The query uses a common table expression (CTE) to first rank the records based on the order_date within each partition defined by customer_id and product_name. Only the latest record for each unique combination is selected.

#### Common Mistakes
Forgetting to include a timestamp or version column if one isn't available in your data.
Misordering the rows during window function application, leading to incorrect results.
Additional Considerations
If multiple records have the same order_date, you may need additional logic to handle these cases. One approach could be using RANK() instead of ROW_NUMBER() which assigns the same rank to tied entries.
Ensure that your table schema and data types are correctly defined before running such queries.

**Comparison:**

| order_id | customer_id | product_name | quantity | order_date |
|----------|-------------|--------------|----------|------------|
| 2        | 101         | Laptop       | 2        | 2023-07-01 |
| 4        | 102         | Smartphone   | 1        | 2023-06-15 |

**Code:**

```sql
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        product_name,
        quantity,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id, product_name ORDER BY order_date DESC) as rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    product_name,
    quantity,
    order_date
FROM ranked_orders
WHERE rn = 1;
```

**Example input / output:**

```sql
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        product_name,
        quantity,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id, product_name ORDER BY order_date DESC) as rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    product_name,
    quantity,
    order_date
FROM ranked_orders
WHERE rn = 1;
order_id	customer_id	product_name	quantity	order_date
2	101	Laptop	2	2023-07-01
4	102	Smartphone	1	2023-06-15
```

**Data:**

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_name VARCHAR(255),
    quantity INT,
    order_date DATE
);

-- Sample data
INSERT INTO orders (order_id, customer_id, product_name, quantity, order_date) VALUES
(1, 101, 'Laptop', 1, '2023-06-01'),
(2, 101, 'Laptop', 2, '2023-07-01'),
(3, 102, 'Smartphone', 1, '2023-05-15'),
(4, 102, 'Smartphone', 1, '2023-06-15');
```

**Flow:**

1. Identify Key Columns
2. Add Timestamp Column (ifnecessary)
3. Apply ROW_NUMBER WindowFunction
4. Filter Rows with Row Number1
5. Return Latest Records Only

