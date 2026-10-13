---
type: "practice"
difficulty: "Deep"
---

# Your warehouse merge step is the bottleneck, not the extraction. How do you tune destination-side ingestion performance?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Managed Ingestion and Re-syncs](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address the bottleneck in your data warehouse's merge step, we need to optimize the destination-side ingestion process. This involves understanding the current setup, identifying bottlenecks, and applying various strategies such as parallelization, indexing, and partitioning.

#### Triage

The question is focused on optimizing the performance of the data warehouse’s merge step by tuning the destination-side ingestion process. The key aspects being probed are:

Current setup and configuration of the data pipeline.
Identifying specific bottlenecks in the merge step.
Strategies to improve performance, including parallelization, indexing, and partitioning.

#### First-Principles Solution
1. Understand the Current Setup

First, review the current Airbyte or Fivetran configuration:

Source connectors: What sources are being ingested, and what is their volume?
Destination database: Type (e.g., PostgreSQL, Redshift), schema design, and existing indexes.
Merge strategy: How data conflicts are handled during merging.
2. Identify Bottlenecks

Bottlenecks can be:

Disk I/O: High write operations can cause disk contention.
Network Latency: Slow network connections between the source and destination.
Concurrency Limits: Limited concurrency in writes to the database.
Indexing: Poorly designed indexes that slow down merge operations.
3. Optimize Merge Operations

To mitigate these bottlenecks, we can:

Parallelize Writes: Increase the number of concurrent write operations to reduce the total time taken for merges.
Partitioning: Use partitioned tables to distribute writes more evenly across disks and processes.
Index Management: Ensure indexes are optimized to support efficient merge operations.
4. Implement Strategies

a) Parallelization

Adjust Airbyte or Fivetran configuration to use multiple worker threads for writing data.

#### Cons
ider using a distributed database like Redshift with its own parallel processing capabilities.

b) Partitioning

Create partitioned tables based on relevant columns (e.g., date, ID).

#### Example
CREATE TABLE sales_data (id BIGINT, date DATE, amount DECIMAL) PARTITION BY RANGE (date);

c) Index Management

Remove unnecessary indexes that slow down write operations.
Rebuild and optimize existing indexes to reduce fragmentation.
5. Monitor and Test
Set up monitoring tools to track performance metrics (e.g., write latency, disk usage).
Run load tests to ensure the new configuration meets performance targets.

#### Closing Summary

By understanding the current setup, identifying bottlenecks, and implementing strategies such as parallelization, partitioning, and optimized indexing, we can significantly improve the performance of the merge step in your data warehouse. Monitoring and testing are crucial for ensuring the solution works effectively.

#### Common Mistakes/Follow-Ups
Ignoring Source Data Volume: Failing to consider the volume of source data can lead to ineffective optimization.
Over-indexing: Adding too many indexes without considering their impact on write operations.
Inadequate Testing: Not thoroughly testing new configurations under load can result in suboptimal performance.

#### Triage

Would you like more details on any specific aspect, such as configuring Airbyte/Fivetran for parallel writes or partitioning strategies?

**Comparison:**

| Worker | Time Spent (seconds) |
|--------|---------------------|
| 1      | 60                  |
| 2      | 35                  |
| 4      | 25                  |
| 8      | 20                  |

**Code:**

```sql
-- Example of creating a partitioned table
CREATE TABLE sales_data (
    id BIGINT,
    date DATE,
    amount DECIMAL
) PARTITION BY RANGE (date);

-- Example of adding a new partition
ALTER TABLE sales_data ADD partitions (partition2023_01 FOR VALUES FROM ('2023-01-01') TO ('2023-01-31'));
```

**Example input / output:**

```text
Input (Airbyte/Fivetran configuration):
{
  "source": {
    "type": "Postgres",
    "url": "jdbc:postgresql://localhost:5432/source_db"
  },
  "sink": {
    "type": "Redshift",
    "url": "jdbc:redshift://redshift-cluster-1.abc123.region.redshift.amazonaws.com:5439/dev"
  },
  "settings": {
    "max_workers": 1,
    "batch_size": 1000
  }
}

Output (Optimized configuration):
{
  "source": {
    "type": "Postgres",
    "url": "jdbc:postgresql://localhost:5432/source_db"
  },
  "sink": {
    "type": "Redshift",
    "url": "jdbc:redshift://redshift-cluster-1.abc123.region.redshift.amazonaws.com:5439/dev"
  },
  "settings": {
    "max_workers": 8,
    "batch_size": 1000
  }
}
```

**Data:**

```json
{
  "source": {
    "type": "Postgres",
    "url": "jdbc:postgresql://localhost:5432/source_db"
  },
  "sink": {
    "type": "Redshift",
    "url": "jdbc:redshift://redshift-cluster-1.abc123.region.redshift.amazonaws.com:5439/dev"
  }
}
```

**Flow:**

1. Yes
2. No
3. Review Current Setup
4. Identify Bottlenecks
5. Are there multiple sources?
6. Parallelize Writes
7. Optimize Indexing
8. Monitor Performance
9. Test Configuration
10. Merge Multiple SourcesParallelly
11. Rebuild and Optimize Indexes

