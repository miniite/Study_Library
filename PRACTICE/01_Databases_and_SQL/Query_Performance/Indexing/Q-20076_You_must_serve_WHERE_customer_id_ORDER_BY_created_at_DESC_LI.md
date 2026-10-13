---
type: "practice"
difficulty: "Intermediate"
---

# You must serve WHERE customer_id = ? ORDER BY created_at DESC LIMIT 50 and also a rare full history export for the same customer. Design the index and explain how both queries use it.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keyset Pagination](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively handle both types of queries—serving individual customers' recent activities and exporting their entire history—we need to design an appropriate indexing strategy that balances performance and storage efficiency. The key idea is to create a composite index on customer_id and created_at, which will allow us to efficiently retrieve the latest records for each customer while also supporting full history exports without excessive overhead.

#### Triaging the Question

The question requires designing an index that can support two distinct types of queries:

Serving individual customers' recent activities (WHERE customer_id = ? ORDER BY created_at DESC LIMIT 50).
Exporting a customer's entire history.

To achieve this, we need to consider the distribution of data and query patterns, ensuring that our indexing strategy minimizes costs for both types of operations while maintaining acceptable performance.

#### First Principles Solution
Composite Index Design:
Create an index on customer_id (as the leading column) and created_at in descending order.
Query Execution Plan:
For serving recent activities, this index will allow us to quickly locate the relevant customer's records and then sort them by created_at.
For exporting full history, this same index can be used, but we might need to consider additional optimizations.

#### Implementation
Steps
Index Creation:
SQL
CREATE INDEX idx_customer_history ON my_table (customer_id DESC, created_at DESC);
Query Analysis and Optimization:
For serving recent activities:
SQL
EXPLAIN SELECT * FROM my_table WHERE customer_id = 123 ORDER BY created_at DESC LIMIT 50;
This query should use the idx_customer_history index efficiently.
For exporting full history:
SQL
EXPLAIN SELECT * FROM my_table WHERE customer_id = 123;
While this query benefits from the same index, we might need to ensure that no additional overhead is introduced due to sorting or other operations.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By creating a composite index on customer_id and created_at, we can effectively support both recent activity serving and full history exports. However, it's crucial to monitor the actual query performance and consider further optimizations if necessary.

#### Common Mistakes
Ignoring the leading column in the index: The customer_id should be the first column to ensure fast lookups by customer.
Overcomplicating the index with unnecessary columns or conditions that could degrade performance for both queries.

#### Example
Input->Output

For serving recent activities:

SQL
EXPLAIN SELECT * FROM my_table WHERE customer_id = 123 ORDER BY created_at DESC LIMIT 50;

The EXPLAIN plan should show an index scan on idx_customer_history.

For exporting full history:

SQL
EXPLAIN SELECT * FROM my_table WHERE customer_id = 123;

Again, the EXPLAIN plan should indicate efficient use of idx_customer_history.

Self-Check

The design and explanation cover both query types effectively with a single index strategy.

**Comparison:**

| Customer ID | Created At       | Activity Data         |
|-------------|------------------|-----------------------|
| 123         | 2023-10-06T15:30 | second activity      |
| 123         | 2023-10-05T14:30 | first activity       |

**Code:**

```sql
-- Index creation
CREATE INDEX idx_customer_history ON my_table (customer_id DESC, created_at DESC);
```

**Example input / output:**

```text
-- Serving recent activities (EXPLAIN plan)
EXPLAIN SELECT * FROM my_table WHERE customer_id = 123 ORDER BY created_at DESC LIMIT 50;
-- Expected output:
-- Index Scan using idx_customer_history on my_table

-- Exporting full history (EXPLAIN plan)
EXPLAIN SELECT * FROM my_table WHERE customer_id = 123;
-- Expected output:
-- Index Scan using idx_customer_history on my_table
```

**Data:**

```json
{
  "my_table": {
    "columns": ["id", "customer_id", "created_at", "activity_data"],
    "data": [
      {"id": 1, "customer_id": 123, "created_at": "2023-10-05T14:30:00Z", "activity_data": "first activity"},
      {"id": 2, "customer_id": 123, "created_at": "2023-10-06T15:30:00Z", "activity_data": "second activity"},
      // More data
    ]
}
```

**Flow:**

1. Customer Query
2. Index Scan usingidx_customer_history
3. Serves Recent Activities
4. Exports Full History
5. Efficient Index Utilization

