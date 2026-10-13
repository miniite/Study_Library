---
type: "practice"
difficulty: "Expert"
---

# A principal panel: you're told a critical join between two 500M-row tables periodically flips to a catastrophic plan after data reloads. Beyond fixing this instance, architect the org-wide defense against plan instability from bulk reloads (statistics locking, plan baselining, reload sequencing).

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

Database indexing and query optimization are crucial for maintaining performance in large-scale systems. In this scenario, a critical join operation between two 500M-row tables is experiencing plan instability after periodic data reloads. Our goal is to architect a robust solution that not only addresses the current issue but also establishes an organization-wide defense against such instabilities.

#### Triaging the Question

The question asks us to resolve a specific performance issue and then design a comprehensive strategy to prevent similar issues in the future. The key aspects are understanding why the plan flips, ensuring consistent execution plans post-reload, and implementing preventive measures across all critical queries and tables.

#### Assumptions
We are using PostgreSQL, which is known for its complex query optimization process.
The join operation involves two large tables that are frequently updated through bulk data reloads.
The current plan instability occurs due to changes in statistics after the reload, leading to suboptimal execution plans.

#### First Principles Solution

To address the issue and build a long-term defense mechanism, we need to focus on three main areas: maintaining consistent statistics, implementing query optimization baselines, and managing data reloading sequences.

#### Cons
istent Statistics Maintenance
Query Optimization Baselines
Data Reload Sequencing
Step-by-Step Solution

1. Consistent Statistics Maintenance Post-reload statistics can significantly impact the planner's decision-making process. To ensure consistent performance, we need to maintain up-to-date and accurate statistics.

Use AUTOVACUUM and ANALYZE Regularly: Ensure that the database automatically collects statistics with AUTOVACUUM and runs full or incremental ANALYZE after bulk data reloads.
Materialized Views for Large Joins: For frequently queried joins, consider using materialized views. Materialized views can precompute join results, ensuring consistent execution plans.

2. Query Optimization Baselines To prevent plan instability due to changes in statistics, we should establish query optimization baselines.

Plan Caching and Pinning: Implement a mechanism to cache and pin optimal execution plans for critical queries. This ensures that the same execution plan is used even if statistics change.
Rule-Based Optimizer (RBO) vs Cost-Based Optimizer (CBO): Leverage RBO settings when appropriate, as it can provide deterministic results by prioritizing certain rules.

3. Data Reload Sequencing To manage data reloads effectively and minimize the impact on performance, we need to sequence them carefully.

Isolated Data Reloads: Schedule bulk data reloads at off-peak times or isolate specific tables for reloading.
Incremental Reloads: Where possible, use incremental reloads instead of full reloads. This reduces the risk of statistics changes affecting multiple queries simultaneously.
Parallelism Management: Control the level of parallelism during bulk inserts to avoid overwhelming the database.

#### Closing Summary

By implementing consistent statistics maintenance, establishing query optimization baselines, and carefully managing data reload sequences, we can mitigate the risk of plan instability. This approach ensures that critical queries maintain optimal performance even after large-scale data reloads.

#### Common Mistakes/Follow-Ups
Over-relying on ANALYZE: Frequent manual ANALYZE runs can lead to excessive I/O and overhead.
Ignoring Materialized Views: Not considering materialized views for frequently used join operations might result in suboptimal query performance.
Inadequate Plan Caching: Failing to cache execution plans can lead to repeated recompilation of the same queries, causing performance degradation.
Triaging the Follow-Up Questions
How would you handle a scenario where multiple tables are involved and their statistics change independently?
What are the trade-offs between using AUTOVACUUM and running manual ANALYZE?
Can you provide an example of how to implement plan pinning in PostgreSQL?

**Code:**

```

-- Example of a materialized view for a critical join operation CREATE MATERIALIZED VIEW sales_summary AS SELECT s.sale_id, p.product_name, SUM(s.quantity) as total_quantity_sold FROM sales_table s JOIN products_table p ON s.product_id = p.product_id GROUP BY s.sale_id, p.product_name;

CODE
Copy

```sql
-- Example of pinning a query execution plan using rule-based optimization (RBO)
SET enable_rule_based_optimizer TO true; -- Enable RBO for deterministic plans
PREPARE query_plan AS SELECT * FROM sales_summary WHERE product_name = 'Widget';
EXECUTE query_plan;
```

**Example input / output:**

```text
Input:
- Schedule a data reload of the `sales_table` and `products_table`.
- Monitor the execution plan post-reload.

Output:
- Verify that the join operation uses an optimal execution plan.
- Confirm that statistics are updated correctly after reloads.
```

**Flow:**

1. Yes
2. No
3. Initiate Data Reload
4. Is Table Large?
5. Run Incremental ANALYZE
6. Maintain Materialized Views ifNecessary
7. Run Full ANALYZE
8. Monitor Plan Stability
9. Implement Query OptimizationBaselines
10. Sequence Data Reloads forCritical Tables

