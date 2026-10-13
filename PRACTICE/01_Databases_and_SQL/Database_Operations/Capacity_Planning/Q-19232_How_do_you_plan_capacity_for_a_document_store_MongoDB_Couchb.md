---
type: "practice"
difficulty: "Deep"
---

# How do you plan capacity for a document store (MongoDB/Couchbase) where document size growth, working-set fit, and oplog window each constrain scale differently?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Capacity Planning and Throughput](<../../../../01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When planning capacity for a document store like MongoDB or Couchbase, we need to consider several factors that affect scalability, including document size growth, working set fit, and oplog window. These constraints interact in complex ways, requiring us to make strategic decisions based on the specific workload requirements.

The core idea is to understand the underlying data dynamics, estimate capacity needs accurately, and design a scalable architecture that can handle the expected load without bottlenecks or performance degradation.

First, let's triage the key questions:

Document Size Growth: This affects storage requirements and the overall size of the database.
Working Set Fit: The amount of data frequently accessed at any given time influences cache efficiency and query performance.
Oplog Window: In MongoDB, this affects replication latency and consistency needs.

Next, let's break down each factor:

Document Size Growth

Document sizes can grow over time due to increased metadata or longer logs. To handle this, we need:

Estimate the size growth rate per document.
Calculate total storage requirements: Multiply the average document size by the expected number of documents and account for growth.

For example, if you expect a 10% monthly increase in document size, and your current average is 1 MB, over a year it will grow to approximately 2.65 MB per document.

Working Set Fit

The working set (the data that needs to be kept in RAM) affects cache performance:

Determine the working set size: This depends on the most frequently accessed documents and their sizes.
Calculate cache efficiency: Higher working sets lead to better hit rates, but require more memory. Optimize by tuning the cache size to balance between hit rate and memory usage.

For instance, if 20% of your documents are part of the working set, and each is 1 MB, you need at least 2 GB in RAM for this subset.

Oplog Window

In MongoDB, the oplog window affects how up-to-date secondary replicas are:

Define consistency needs: Decide whether eventual or strong consistency is required.
Adjust oplog size: Larger oplogs allow more time between primary and replica updates but consume more storage. For example, a 1-hour oplog might require 2 GB of disk space.

For a 4-node replica set with a 1-hour oplog window, the total additional storage needed is approximately 8 GB (assuming each node needs its own copy).

#### First-Principles Solution

To plan capacity effectively:

Estimate Initial Requirements: Start by estimating the initial document size and working set.
Scale Out Early: Design your database to scale out horizontally from the start, using sharding or partitioning strategies.
Monitor and Optimize: Continuously monitor performance metrics such as IOPS, network usage, and memory pressure. Use this data to make informed decisions about scaling.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Proper capacity planning requires balancing storage, caching, and replication strategies to meet application needs while keeping costs under control. Key mistakes include:

Over-provisioning: Allocating too much storage can lead to unnecessary expenses.
Underestimating growth: Ignoring gradual document size increases can result in performance issues later.

#### Follow-ups
might include discussing specific metrics (like IOPS, latency) or exploring advanced features like read preferences or automatic scaling mechanisms.

**Comparison:**

| Metric | Value           |
|--------|----------------|
| Current Average Doc Size | 1 MB        |
| Monthly Growth Rate      | 0.1         |
| Working Set Percentage   | 20%         |
| Cache Size               | 2 GB        |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Document Size Growth] --> B(Storage Calculation);
    C[Working Set Fit] --> D(Cache Tuning);
    E[Oplog Window] --> F(Replication Configuration);
    G[Estimate Initial Requirements] --> H[Scale Out Early];
    H --> I[Monitor and Optimize]
```

**Example input / output:**

```text
Initial Document Size: 1 MB
Working Set Percentage: 20%
Oplog Window (hours): 1

Storage Requirements:
- Current Documents: 5,000
- Estimated Future Documents: ~7,893 (using growth rate)

Cache Configuration:
- Working Set Size: ~1 GB (20% of total documents * 1 MB)
- Cache Size: 2 GB

Replication Setup:
- Oplog Window: 1 hour
- Storage Needed for Oplog: 2 GB per node (4 nodes = 8 GB total)

Total Initial Storage Requirement: 13.8 GB + 8 GB (Oplog) = 21.8 GB
```

**Data:**

```json
{
  "documentSizes": {
    "currentAverage": 1024, // in KB
    "monthlyGrowthRate": 0.1
  },
  "workingSetSizePercentage": 20,
  "cacheSizeGB": 2
}
```

**Flow:**

1. Estimate Initial Document Sizes
2. Calculate Storage Needs
3. Tune Cache Configuration
4. Configure Oplog Window
5. Plan Scale Out Strategy
6. Monitor and OptimizePerformance

