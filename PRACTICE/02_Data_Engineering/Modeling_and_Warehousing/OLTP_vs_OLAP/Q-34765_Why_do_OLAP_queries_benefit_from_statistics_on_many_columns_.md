---
type: "practice"
difficulty: "Deep"
---

# Why do OLAP queries benefit from statistics on many columns while OLTP planners lean heavily on a few high-selectivity indexed columns — how does that shape ANALYZE strategy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Physical Design for OLTP vs OLAP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Physical_Design_for_OLTP_vs_OLAP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this discussion, we'll explore why OLAP (Online Analytical Processing) queries benefit from statistics on many columns while OLTP (Online Transactional Processing) systems heavily rely on a few high-selectivity indexed columns. This understanding will help us optimize the ANALYZE strategy in database management.

#### Triaging the Question

The question is essentially about how query optimization differs between OLAP and OLTP systems, with specific emphasis on the role of statistics and indexing. The interviewer is probing our knowledge of:

Statistics Use: How different types of queries benefit from various statistical information.
Indexing Strategy: Why certain columns are more critical for performance in each scenario.
Optimization Techniques: Understanding when and why to collect specific types of data.

#### First-Principles Solution
OLAP Queries: Broad and Aggregated

OLAP systems handle complex, aggregated queries that often involve multiple dimensions and large datasets. These queries require accurate statistical information across many columns to estimate the distribution of data accurately. This is because:

Aggregation: OLAP queries frequently aggregate data over wide ranges, necessitating understanding how values are distributed.
Multiple Dimensions: Queries may involve cross-tabulations or multi-dimensional analysis, requiring comprehensive statistics for efficient execution.
OLTP Queries: Narrow and Transactional

OLTP systems handle a high volume of transactions in real-time with low latency requirements. They focus on specific records, often using indexed columns to quickly locate them. This strategy relies heavily on:

High-Selectivity Indexes: These allow rapid access to the exact row or set of rows needed for a transaction.
Few Critical Columns: High-selectivity indexes are placed on columns that are frequently used in WHERE clauses, ensuring quick lookup times.
Analyzing the ANALYZE Strategy
Collecting Statistics

OLAP Systems should collect statistics across many columns to support complex queries involving multiple dimensions and aggregations.

#### Example
A business intelligence system needs accurate counts and distributions of customer segments for reporting purposes.

OLTP Systems should focus on high-selectivity indexes on critical columns used in WHERE clauses.

#### Example
An e-commerce application frequently searches by product ID or user ID; these columns would be highly selective, reducing the number of rows to scan.

#### Trade-offs
and Practical Considerations
Storage Overhead: Collecting statistics for many columns can consume storage resources, but this is acceptable in OLAP systems due to their nature.
Update Frequency: High-frequency updates are more frequent in OLTP systems, so statistics need to be updated regularly to remain accurate.
Query Performance vs Maintenance: OLAP queries benefit from slower maintenance of comprehensive statistics because the cost is spread over multiple executions.

#### Conclusion
and Common Mistakes
Overcollecting Statistics: It’s a mistake to collect unnecessary statistics in OLTP systems, which can lead to inefficiencies.
Underindexing Critical Columns: Failing to index high-selectivity columns in OLTP systems can result in poor query performance.
Ignoring Maintenance: Neglecting regular maintenance of statistics and indexes can degrade overall system performance.

#### Closing Summary

The key difference lies in the nature of queries: OLAP systems benefit from broad statistical information, while OLTP systems prioritize fast access to specific rows via high-selectivity indexing. This understanding helps in designing effective ANALYZE strategies that optimize query performance without compromising on storage and maintenance costs.

**Comparison:**

```
Column	Type
customer_id	Cardinality
segment	Histogram
```

**Code:**

```sql
-- Example of collecting statistics in PostgreSQL
ANALYZE sales_data;
```

**Example input / output:**

```text
Input:
ANALYZE sales_data;

Output:
Statistics collected for 'segment' and 'customer_id'.
```

**Data:**

```json
{
  "columns": ["customer_id", "product_id", "segment", "price", "quantity"],
  "statistics_needed": [
    {"column": "segment", "type": "histogram"},
    {"column": "customer_id", "type": "cardinality"}
  ]
}
```

**Flow:**

1. Many Columns
2. Few Columns
3. Yes
4. No
5. OLAP Query
6. Use Statistics?
7. Accurate Aggregation
8. High-Selectivity Indexes
9. OLTP Query
10. Index Critical Columns?
11. Fast Lookup Performance
12. Poor Query Performance

