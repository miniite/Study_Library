---
type: "practice"
difficulty: "Deep"
---

# How do you decide the clustering/sort key for an OLAP fact table when queries filter mostly by date but sometimes by customer — and why can't you just index like on OLTP?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Physical Design for OLTP vs OLAP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Physical_Design_for_OLTP_vs_OLAP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When designing an OLAP (Online Analytical Processing) fact table, especially one that frequently queries data by date but occasionally by customer, choosing a clustering/sort key is crucial to optimize performance. The goal is to balance query performance with storage efficiency and update frequency. Clustering keys should be chosen based on the primary access patterns, while avoiding excessive indexing as seen in OLTP systems.

#### Triaging the Question

The question revolves around optimizing the clustering/sort key for an OLAP fact table where most queries filter by date but occasionally by customer. The challenge is to balance the benefits of clustering and avoid unnecessary overhead like excessive indexing found in OLTP systems, which can degrade performance during updates.

#### Assumptions
Primary Access Patterns: Queries predominantly filter by date.
Occasional Customer Filtering: Queries that involve filtering by customer are less frequent but still important.
Storage and Performance Requirements: The system needs to handle large volumes of data with efficient query execution while maintaining reasonable storage costs.
Step-by-Step Solution

Understanding Clustering/Sorting in OLAP:

In OLAP systems, clustering keys help organize the data physically on disk such that rows with similar values are stored close together.
This reduces I/O operations and improves query performance by allowing the database to skip over large portions of data.

Choosing the Primary Clustering Key (Date):

Since most queries filter by date, setting date as the clustering key is a logical choice. This ensures that date-based queries can quickly locate relevant rows.
SQL
CREATE TABLE fact_table (
    id INT,
    customer_id INT,
    transaction_date DATE,
    amount DECIMAL(10, 2),
    ...
    PRIMARY KEY (id),
    CLUSTERING BY transaction_date DESC
);

Handling Occasional Customer Queries:

While date is the primary clustering key, we need to ensure that customer-related queries can still perform efficiently.
We can use a composite clustering key combining both date and customer ID:
SQL
CREATE TABLE fact_table (
    id INT,
    customer_id INT,
    transaction_date DATE,
    amount DECIMAL(10, 2),
    ...
    PRIMARY KEY (id),
    CLUSTERING BY transaction_date DESC, customer_id ASC
);
This ensures that date-based queries are fast and customer-specific queries can be optimized by scanning within a date range.

Avoiding Excessive Indexes:

OLTP systems often use multiple indexes to speed up updates and reads. In contrast, OLAP systems typically benefit more from clustering keys.
Avoid creating additional indexes on customer_id or other fields unless absolutely necessary. This helps reduce storage overhead and maintenance costs.

#### Cons
ideration of Storage and Performance Trade-offs:

Balancing between storage efficiency and performance is critical. Clustering by date ensures that date-based queries are fast, but frequent customer queries should still be optimized.
Monitor query performance using tools like Explain Plans to ensure that the chosen clustering strategy meets requirements.

#### Closing Summary

Choosing a clustering/sort key for an OLAP fact table involves understanding primary access patterns and optimizing for both date-based and occasional customer-based queries. By setting transaction_date as the primary clustering key and adding customer_id for secondary optimization, we can achieve efficient query performance without unnecessary overhead.

#### Common Mistakes
and Follow-ups

Mistake 1: Choosing too many indexes or secondary keys in OLAP systems, leading to increased storage costs and maintenance.

#### Follow-up
Always validate the chosen clustering strategy with real-world data using performance monitoring tools.

Mistake 2: Ignoring occasional customer queries, which can lead to suboptimal performance during critical operations.

#### Follow-up
Ensure that customer-specific queries are optimized by considering composite keys or additional filtering mechanisms.

**Comparison:**

| id | customer_id | transaction_date | amount |
|----|------------|------------------|--------|
| 1  | 42         | 2023-10-01       | 50.99  |
| 2  | 43         | 2023-10-02       | 60.75  |

**Code:**

```sql
-- Example OLAP Table with Composite Clustering Key
CREATE TABLE fact_table (
    id INT,
    customer_id INT,
    transaction_date DATE,
    amount DECIMAL(10, 2),
    ...
    PRIMARY KEY (id),
    CLUSTERING BY transaction_date DESC, customer_id ASC
);
```

**Example input / output:**

```text
-- Example Query for Date-based Filter
SELECT * FROM fact_table WHERE transaction_date BETWEEN '2023-09-01' AND '2023-10-31';

-- Example Query for Customer-based Filter within a Date Range
SELECT * FROM fact_table WHERE customer_id = 42 AND transaction_date BETWEEN '2023-09-01' AND '2023-10-31';
```

**Data:**

```json
{
    "id": 1,
    "customer_id": 42,
    "transaction_date": "2023-10-01",
    "amount": 50.99,
    ...
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is date filter predominant?
5. Use date as clustering key
6. Consider composite key (date,customer_id)
7. Monitor and optimize queries
8. Optimize for customer-specificqueries if necessary
9. End

