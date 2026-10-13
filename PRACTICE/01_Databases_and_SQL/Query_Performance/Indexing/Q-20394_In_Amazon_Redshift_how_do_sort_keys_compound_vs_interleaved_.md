---
type: "practice"
difficulty: "Deep"
---

# In Amazon Redshift, how do sort keys (compound vs interleaved) act like a lightweight index, and when does an interleaved sort key hurt?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Random Keys and Index Locality](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Random_Keys_and_Index_Locality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how sort keys in Amazon Redshift behave as lightweight indexes, particularly distinguishing between compound and interleaved sort keys, we need to delve into their mechanics and implications on query performance. This explanation will cover the basic concepts, the practical use cases for each type of sort key, and when using an interleaved sort key might lead to suboptimal results.

Firstly, in Amazon Redshift, sort keys are a form of metadata that helps organize data within tables to optimize query performance. They act as lightweight indexes by allowing Redshift to quickly locate the relevant data slices during query execution. The two main types of sort keys are compound and interleaved, each with distinct behaviors and use cases.

Compound sort keys involve specifying multiple columns in order of their importance for sorting. Redshift uses this information to ensure that rows with similar values on the leading column(s) are stored close together, which can significantly speed up queries that filter or join on those leading columns.

Interleaved sort keys work differently by storing data in a different manner. Instead of clustering rows based on the leading column value, interleaved sort keys mix data from multiple partitions to spread out the data more evenly across slices. This approach is designed to reduce skew and improve parallelism but can sometimes lead to less efficient query execution.

To illustrate this, let's consider an example where we have a large fact table with columns date, region, and sales. If we use a compound sort key on (date, region), Redshift will store rows from the same date and region together, which is ideal for queries that filter by both date and region. On the other hand, using an interleaved sort key might be beneficial if there's significant skew in the data distribution of region, ensuring that no single region dominates a slice.

However, when should we avoid using interleaved sort keys? Consider a scenario where our query heavily relies on filtering by date but doesn't have any specific pattern for region. In such cases, an interleaved sort key could spread out the date ranges across slices, leading to more extensive scans and reducing overall performance. Similarly, if the data is naturally skewed (e.g., many sales from a single region), an interleaved approach might not effectively mitigate this skew, causing suboptimal query execution.

In summary, understanding the behavior of sort keys in Amazon Redshift helps us make informed decisions about indexing strategies. Compound sort keys are generally more predictable and can lead to faster query performance if the queries align well with the key columns. Interleaved sort keys offer a different strategy to handle data skew but come with potential drawbacks when the query patterns don't match.

**Comparison:**

| sales_fact               | orders_dim                |
| date, region, sales      | order_id, customer_name, order_date |
| SORTKEY(date, region)    | INTERLEAVE KEY(order_date)   |

**Code:**

```sql
-- Example of creating a table with compound sort keys
CREATE TABLE sales_fact (
    date DATE,
    region VARCHAR(50),
    sales DECIMAL,
    PRIMARY KEY (date, region)
) SORTKEY(date, region);

-- Example of using interleaved sort keys
CREATE TABLE orders_dim (
    order_id INT,
    customer_name VARCHAR(100),
    order_date DATE,
    PRIMARY KEY (order_id)
) INTERLEAVE KEY(order_date);
```

**Example input / output:**

```text
-- Query using compound sort key
SELECT * FROM sales_fact WHERE date = '2023-01-01' AND region = 'North';

-- Expected output:
+------------+----------+---------+
| date       | region   | sales   |
+------------+----------+---------+
| 2023-01-01 | North    | 100     |
| 2023-01-01 | North    | 150     |
+------------+----------+---------+

-- Query using interleaved sort key
SELECT * FROM orders_dim WHERE order_date > '2023-01-01';

-- Expected output:
+-----------+---------------+------------+
| order_id  | customer_name | order_date |
+-----------+---------------+------------+
| 2         | Bob           | 2023-01-02 |
| 3         | Charlie       | 2023-01-03 |
+-----------+---------------+------------+
```

**Data:**

```json
{
  "sales_fact": {
    "date": ["2023-01-01", "2023-01-02", "2023-01-01"],
    "region": ["North", "South", "North"],
    "sales": [100, 200, 150]
  },
  "orders_dim": {
    "order_id": [1, 2, 3],
    "customer_name": ["Alice", "Bob", "Charlie"],
    "order_date": ["2023-01-01", "2023-01-02", "2023-01-03"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query Execution withCompound Sort Key
6. Is the query filtering on dateand region?
7. Efficient query performancedue to data clustering
8. Slower query performance;may require full scan
9. Query Execution withInterleaved Sort Key
10. Is there significant skew inorder_date?
11. Interleaving mitigates skew,improving parallelism
12. Skew not effectively handled;potential for suboptimalperformance

