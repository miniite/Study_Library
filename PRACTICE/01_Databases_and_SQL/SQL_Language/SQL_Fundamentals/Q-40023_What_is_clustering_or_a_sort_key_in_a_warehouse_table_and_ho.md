---
type: "practice"
difficulty: "Deep"
---

# What is clustering or a sort key in a warehouse table, and how does it help range-filter queries on a large fact table?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Large Values and TOAST](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Clustering and sort keys are essential concepts for optimizing query performance in data warehouses, particularly when dealing with large fact tables that require frequent range filters. At a high level, clustering involves organizing rows of a table based on specific columns, while a sort key is a column or combination of columns used to store the data in a particular order. The primary benefit of these techniques is their ability to significantly speed up query execution by reducing the amount of data that needs to be scanned.

To understand how clustering and sort keys help with range-filter queries, let's first define what they are and then delve into their practical applications.

**Comparison:**

```
Date Range	Scanned Rows (Without Clustering/Sort Key)	Scanned Rows (With Clustering on sale_date)
2023-01-01 to 2023-06-30	Approx. 50% of the table	Approximately 5% of the table
```

**Code:**

```sql
-- Example table structure in a data warehouse
CREATE TABLE sales (
    sale_id INT,
    product_id INT,
    customer_id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
);
```

**Example input / output:**

```
Example Queries

Without Clustering or Sort Keys

Request:
SQL
Copy
SELECT * FROM sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-06-30';
Response: The database engine may need to scan a large portion of the table, which can be inefficient if the date range covers a significant part of the data.

With Clustering on sale_date

Request:
SQL
Copy
SELECT * FROM sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-06-30';
Response: The database can use the clustering index to quickly locate and retrieve only the rows within the specified date range, significantly reducing the amount of data scanned.

With a Sort Key on sale_date

Request:
SQL
Copy
SELECT * FROM sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-06-30';
Response: Similar to clustering, the sort key allows for efficient range queries by organizing rows in a specific order, thus minimizing the scan and improving performance.
```

**Data:**

```text
The `sales` table has columns such as `sale_id`, `product_id`, `customer_id`, `sale_date`, and `amount`. A common requirement is to query sales data within a specific date range.
```

**Flow:**

1. Performance Implications
2. Query without clustering/sortkey
3. Scans entire table
4. Query with clustering on`sale_date`
5. Locates rows within date range
6. Query with sort key on`sale_date`
7. Locates rows within date rangemore efficiently
8. High CPU usage, slowperformance
9. Reduced scan time, fasterperformance
10. Even better performance dueto sorted order

