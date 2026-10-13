---
type: "practice"
difficulty: "Deep"
---

# How would you design an index so that WHERE tenant_id = ? ORDER BY created_at DESC LIMIT 20 is fast in a multi-tenant app?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keyset Pagination](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize the query WHERE tenant_id = ? ORDER BY created_at DESC LIMIT 20 in a multi-tenant application, we need to design an index that minimizes the cost of filtering by tenant_id, sorting by created_at, and limiting the results. This involves understanding the key aspects: how to filter by tenant, sort efficiently, and apply the limit.

Firstly, let's restate what the question is probing: the interviewer wants to assess your ability to design indexes for complex queries that involve filtering based on a specific value (tenant_id), sorting in descending order (created_at DESC), and limiting results. They are also interested in how you would consider trade-offs between performance, storage, and query complexity.

#### Assumptions
The table contains multiple columns including tenant_id, created_at, and possibly other fields.
There is a significant number of tenants with varying numbers of records.
The application requires efficient execution for each tenant's queries independently.
The database system supports multi-column indexes.
Step-by-Step Solution
1. Index Design

We need to design an index that optimizes the WHERE clause and leverages the ordering. A composite index on (tenant_id, created_at) would be ideal because it allows for efficient filtering followed by sorting in a single pass of the indexed data.

SQL
CREATE INDEX idx_tenant_created ON your_table (tenant_id DESC, created_at DESC);

This index will allow the database to quickly locate records for a specific tenant_id and then sort them efficiently. The order is important here because we need to limit the results after sorting.

2. Query Execution Plan

With this index in place, let's consider how the query would be executed:

Filtering by Tenant ID: The database will use the tenant_id DESC part of the index to quickly narrow down the rows that belong to the specific tenant.
Sorting by Created At: Once filtered, the created_at DESC part ensures that the sorted order is maintained without additional overhead.
3. Limit Application

After filtering and sorting, applying the LIMIT 20 clause will return only the top 20 results from the ordered set of records for that tenant.

Potential Issues and Trade-offs
Index Size: The composite index on (tenant_id DESC, created_at DESC) may consume more space compared to a single-column index. This needs to be balanced against query performance.
Insertion Overhead: When new data is inserted into the table, both tenant_id and created_at need to be updated in the index, which could impact write performance slightly.
Read Performance: The read performance for queries targeting a specific tenant will benefit significantly from this index. However, if there are a large number of tenants with similar activity levels, the cost might not be justified.
Alternative Approaches

Partitioning by Tenant ID: In cases where there are many tenants and each has a significant amount of data, partitioning the table by tenant_id can further optimize read performance.

SQL
CREATE TABLE your_table (
    tenant_id INT,
    created_at TIMESTAMP,
    -- other columns
) PARTITION BY LIST (tenant_id);

Secondary Indexes: If specific queries might require different filtering criteria, additional secondary indexes could be beneficial. For instance, an index on created_at alone if frequently queried by timestamp.

#### Closing Summary

In conclusion, designing the (tenant_id DESC, created_at DESC) composite index is a strategic choice to optimize the given query in a multi-tenant application. This approach leverages the strengths of both filtering and sorting in a single pass, ensuring that read performance is maximized for each tenant's data while minimizing additional overhead.

#### Common Mistakes
and Follow-ups
Forgetting About Tenant ID Filtering: An index without tenant_id would not effectively filter the results.
Incorrect Index Order: Using (created_at DESC, tenant_id DESC) instead of (tenant_id DESC, created_at DESC) would require a separate pass for filtering and sorting.
Ignoring Other Query Scenarios: Not considering other potential query patterns that might benefit from different indexes.

By carefully designing the index and understanding its implications, you can ensure efficient and scalable performance in your multi-tenant application.

