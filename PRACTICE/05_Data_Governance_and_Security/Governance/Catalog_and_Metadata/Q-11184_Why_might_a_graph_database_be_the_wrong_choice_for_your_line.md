---
type: "practice"
difficulty: "Expert"
---

# Why might a graph database be the wrong choice for your lineage/metadata store despite lineage being 'a graph'? What alternatives (relational + adjacency, document, columnar, hybrid) would you weigh?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Lineage Graphs](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Scaling_Lineage_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering a data catalog and metadata engineering for lineage tracking, a graph database might not always be the best choice despite the fact that lineage is fundamentally a graph concept. This is because graph databases excel in managing complex relationships but come with certain trade-offs related to query performance, scalability, and operational complexity. As an expert Data Engineer Architect/Principal, I would weigh several alternatives before making a decision.

Firstly, let's consider why a graph database might not be the optimal choice:

Query Performance: Graph databases are generally optimized for traversing relationships but can suffer in scenarios with complex queries involving multiple hops or large datasets.
Scalability: For very large catalogs, a graph database may require significant resources and optimizations to maintain performance.
Operational Complexity: Managing a graph database involves dealing with distributed systems, indexing strategies, and potential bottlenecks that could complicate operations.

Now, let's explore some alternatives:

Relational Databases

Relational databases (like PostgreSQL) can be very effective for lineage tracking if the relationships are well-defined and not overly complex. They offer ACID compliance, which is crucial for metadata integrity. Here’s a high-level reasoning:

Simplicity: Relational models are well-understood and widely used.
Query Performance: With proper indexing, relational databases can handle complex queries efficiently.
Operational Ease: Easier to manage compared to graph databases.
Adjacency List Models

Adjacency list models store relationships in a way that’s similar to how they would be stored in a graph database but use tables. This approach is simpler and can offer good performance for lineage tracking:

Simplicity: Relatively straightforward schema design.
Flexibility: Can scale horizontally with standard SQL optimizations.
Document Databases

Document databases like MongoDB store data as JSON-like documents, which can be highly flexible and scalable:

Flexibility: Stores relationships in a semi-structured manner, making it easier to evolve the schema over time.
Scalability: Easy to shard and scale horizontally.
Columnar Storage

Columnar storage systems like Apache Parquet are optimized for read-heavy workloads, which is often the case with metadata reads:

Performance: Excellent read performance due to column-wise compression and indexing.
Complexity: Less suited for write-intensive operations but can still manage lineage data effectively.
Hybrid Approaches

Hybrid approaches combine elements of these models. For example, using a graph database as a specialized store for complex relationships while leveraging relational or document databases for the bulk of metadata:

Flexibility and Performance: Combine the strengths of different paradigms.
Complexity: Adds complexity in terms of data consistency and integration.
Summary

While a graph database is theoretically suited for lineage tracking, its limitations in query performance and operational complexity might make it less optimal. Relational databases, adjacency list models, document stores, columnar storage, and hybrid approaches all offer viable alternatives that can be tailored to specific needs.

#### Common mistakes
include:

Overlooking the trade-offs of each model.
Not considering the scalability requirements of the data catalog.
Failing to evaluate the operational complexity introduced by different technologies.

**Comparison:**

| Datasets | Lineages |
|----------|----------|
| SalesData, CustomerInfo | (SalesData -> CustomerInfo) |

**Code:**

```sql
-- Example SQL for a relational lineage tracking schema

CREATE TABLE Dataset (
    id BIGINT PRIMARY KEY,
    name VARCHAR(255),
    version INT
);

CREATE TABLE Lineage (
    source_id BIGINT,
    target_id BIGINT,
    FOREIGN KEY (source_id) REFERENCES Dataset(id),
    FOREIGN KEY (target_id) REFERENCES Dataset(id)
);
```

**Example input / output:**

```text
-- Example SQL queries for lineage tracking

SELECT Dataset.name FROM Lineage JOIN Dataset ON Lineage.target_id = Dataset.id WHERE source_id = 1;

-- Output:
+-----------------+
| name            |
+-----------------+
| CustomerInfo    |
+-----------------+
```

**Data:**

```json
{
  "datasets": [
    {
      "id": 1,
      "name": "SalesData",
      "version": 1
    },
    {
      "id": 2,
      "name": "CustomerInfo",
      "version": 1
    }
  ],
  "lineages": [
    {
      "source_id": 1,
      "target_id": 2
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Dataset
4. Is it a complex query?
5. Use Graph Database
6. UseRelational/Document/Columnar
7. Operational Complexity High
8. Flexibility and PerformanceConsidered
9. Consider Using HybridApproach
10. Evaluate Specific Needs

