---
type: "practice"
difficulty: "Intermediate"
---

# How would you review a teammate's PR that adds a five-column composite index? What questions do you ask about the query it targets before approving?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ JSON and Text Search Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/JSON_and_Text_Search_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When reviewing a PR that introduces a five-column composite index, it’s crucial to ensure both the technical merits and potential implications are thoroughly evaluated. The primary goal is to optimize queries while avoiding unnecessary overheads. Here's how I would approach this task:

Firstly, we need to understand what kind of queries the new index supports. A five-column composite index can be a powerful tool for accelerating specific queries but might not provide much benefit for others.

#### Triage
What Are We Probing?
Query Performance: Do these queries significantly improve performance? Are there any slow queries that this PR addresses?
Indexing Overhead: Will the new index cause excessive write or storage overhead, and is it worth the trade-off?
Index Relevance: Is this a well-chosen combination of columns for indexing purposes?

#### Assumptions
The PR includes performance benchmarks.
The team follows best practices in database design.

#### First-Principles Solution

Understand the Queries:

Review the queries targeted by the index. Identify the most frequently run or slowest-performing ones.
Assess if these queries benefit from a composite index, particularly those with multiple conditions using AND.

Performance Benchmarks:

Compare execution times before and after adding the index for critical queries.
Look at actual performance improvements in terms of latency and throughput.

Indexing Overhead:

Calculate storage and write overheads caused by the new index.
Ensure that the benefit in query speed is proportional to the added costs.

Index Relevance:

Verify if all five columns are necessary for the targeted queries.

#### Cons
ider if any of these columns can be dropped or replaced with more selective ones.

Future Proofing:

Assess whether this index will still be relevant as the schema evolves over time.
Think about potential future queries that might benefit from a different index combination.

#### Closing Summary

Reviewing a PR for a five-column composite index involves evaluating its impact on query performance, storage overhead, and long-term relevance. It’s essential to balance these factors and ensure the changes make sense within the broader context of the application's database architecture.

#### Common Mistakes/Follow-Ups

Over-indexing leading to unnecessary write and read overhead.
Using too many columns in a composite index when some are not selective enough.
Failing to validate performance improvements with real-world data.
Not considering future schema changes that might render the index obsolete.

**Comparison:**

| Index Column     | Type      |
|------------------|-----------|
| customer_id      | INT       |
| product_id       | INT       |
| status           | VARCHAR   |
| order_date       | DATE      |

**Code:**

```sql
-- Example of an SQL query benefiting from a five-column composite index
SELECT * FROM orders WHERE customer_id = 123 AND product_id IN (456, 789) AND status = 'completed' AND order_date BETWEEN '2023-01-01' AND '2023-12-31';
```

**Example input / output:**

```text
- **Input**: A PR that adds a five-column composite index on the `customer_id`, `product_id`, `status`, and `order_date` columns of the `orders` table.
- **Output**:
  - Execution times for critical queries before (e.g., `SELECT * FROM orders WHERE customer_id = 123 AND product_id IN (456, 789) AND status = 'completed'`) and after adding the index.
  - Storage overhead measurements post-index addition.
```

**Data:**

```json
{
    "index_columns": ["customer_id", "product_id", "status", "order_date"],
    "query_frequency": {
        "customer_id = 123": 500,
        "product_id IN (456, 789) AND status = 'completed'": 200
    },
    "storage_overhead": "5 MB",
    "write_overhead": "1%"
}
```

**Flow:**

1. Yes
2. No
3. Review Queries
4. Are these queries slow?
5. Measure Performance Impact
6. Verify Overhead
7. Check Index Relevance
8. Future Proofing
9. Approve PR if benefitsoutweigh costs
10. Reject PR due to overheadconcerns
11. Consider droppingnon-selective columns
12. Final Approval

