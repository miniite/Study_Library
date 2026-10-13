---
type: "practice"
difficulty: "Expert"
---

# How do you set an SLO-driven policy that ties covering-index creation to a query's business criticality tier, so P0 endpoints get bespoke indexes and ad-hoc queries don't?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of setting an SLO-driven policy for advanced indexing in a database environment, we need to align our indexing strategy with business criticality tiers. Specifically, this involves creating bespoke indexes tailored to P0 endpoints while ensuring that ad-hoc queries do not impact performance excessively. The solution hinges on understanding query workloads, defining business SLAs, and implementing an automated index management framework.

#### Triage

The question is essentially asking how to design a policy that prioritizes the creation of covering indexes based on the criticality of the queries they support. P0 endpoints (e.g., mission-critical transactions) should have dedicated indexes, while ad-hoc queries must be managed carefully to avoid excessive resource consumption.

#### Assumptions
The database system supports advanced indexing techniques such as covering and composite indexes.
There are defined SLAs for various business criticality tiers.
We can instrument the database to monitor query performance and resource usage.
There is an automated mechanism to create, maintain, and drop indexes based on metrics.

#### First-Principles Solution

Define Business Criticality Tiers: Start by categorizing queries into different SLA tiers (P0, P1, P2, etc.). For example:

P0 (e.g., critical financial transactions): Require high performance and minimal latency.
P1 (e.g., order processing): Higher priority than ad-hoc but less time-critical.
P2 (e.g., analytics reports): Lower priority, more tolerance for latency.

Identify Query Workloads: Profile the database to understand the types of queries that fall into each tier. This involves analyzing query patterns, execution times, and resource consumption.

Create Covering Indexes: For P0 critical queries, create covering indexes that include all necessary columns in the index. These indexes are designed to minimize I/O operations and improve response times for these high-priority transactions.

Automated Index Management: Implement a mechanism to monitor query performance and automatically adjust indexing strategies based on real-time metrics:

Use database instrumentation tools (e.g., PostgreSQL’s pg_stat_activity or MySQL’s performance schema) to collect data on query execution.
Define rules for when indexes should be created, updated, or dropped. For example, if a P0 query consistently hits an index that is not currently present, automatically create the appropriate covering index.

Ad-hoc Query Management: Ensure ad-hoc queries do not disrupt performance by setting resource limits and prioritizing P0 queries:

Use query costing and scheduling techniques to limit the impact of ad-hoc queries.
Implement a cost-based optimizer that ranks queries based on their criticality and resource consumption.

Monitoring and Feedback Loop: Continuously monitor the effectiveness of the indexing strategy using APM tools like New Relic or Datadog. Adjust policies as needed based on feedback from these metrics.

#### Closing Summary

By combining business SLA tiers with advanced indexing techniques, we can create a robust policy that ensures P0 endpoints receive optimal performance while managing ad-hoc queries to avoid unnecessary resource contention. The key is to automate the process of creating and maintaining indexes based on real-time query workloads and performance data.

#### Common Mistakes/Follow-Ups
Over-indexing: Creating too many indexes can lead to increased maintenance overhead and slower write operations.
Not Considering Query Patterns: Failing to understand the unique characteristics of P0 queries can result in suboptimal indexing strategies.
Resource Contention: Ad-hoc queries not managed properly can cause performance degradation for critical transactions.

Would you like a sample code snippet or further details on any part of this solution?

**Comparison:**

```
SLA Tier	Description	Query Characteristics
P0	Critical transactions	High priority, minimal latency
P1	High-priority business tasks	Higher frequency, moderate resource usage
P2	Non-critical data analysis	Lower priority, higher resource tolerance
```

**Code:**

```sql
-- Example SQL to create a covering index for P0 critical transaction processing

CREATE INDEX idx_p0_transactions_covering ON sales_table (transaction_id, amount, customer_id)
WHERE is_critical_transaction = true;
```

**Example input / output:**

```
Example dry-run input and output:
PLAINTEXT
Copy
Input: A new P0 critical transaction query with high I/O requirements.
Output: Automatically creates a covering index on the sales_table for this type of query to minimize latency.
```

**Data:**

```
Sample schema:
JSON
Copy
{
  "table_name": "sales_table",
  "columns": [
    { "name": "transaction_id", "type": "INT" },
    { "name": "amount", "type": "DECIMAL" },
    { "name": "customer_id", "type": "VARCHAR(255)" },
    { "name": "is_critical_transaction", "type": "BOOLEAN" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Exceeds Threshold
4. Within Threshold
5. Limit Resources
6. Query Execution Profile
7. Is this a critical query?
8. Create Covering Index
9. Monitor Query Performance
10. Adjust Indexes
11. Continue Monitoring
12. Ad-hoc Query Management
13. Monitor and Adjust as Needed

