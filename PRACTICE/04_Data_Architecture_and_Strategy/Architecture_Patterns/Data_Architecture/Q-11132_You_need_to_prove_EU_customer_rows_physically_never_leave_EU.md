---
type: "practice"
difficulty: "Deep"
---

# You need to prove EU customer rows physically never leave EU storage even though a global query engine reads them. Where might a shuffle or result cache accidentally move data across borders, and how do you prevent it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that EU customer rows physically never leave EU storage, we need to architect our system with a robust approach that carefully controls the movement of data, especially during global queries. The key challenges arise from two primary sources: shuffle operations and result caching mechanisms within distributed query engines. Let's break this down step-by-step.

#### Triage

The question asks us to identify where EU customer rows might accidentally move across borders due to shuffle operations or result caching in a global query engine, and then provide solutions to prevent such movements. The goal is to ensure data privacy compliance with GDPR requirements, particularly concerning the physical storage of personal data within the European Union.

Step-by-Step Solution

Understand Shuffle Operations:

Definition: Shuffles are a critical part of distributed computing frameworks like Apache Spark, where data is redistributed across nodes for processing.
Risk: During shuffling, data might be moved to non-EU nodes or cached in intermediate locations.

Identify Common Shuffle Scenarios:

Intermediate Storage: Data may be stored temporarily during the shuffle process before being finalized.
Shuffle Partitions: If a query involves shuffling large partitions of data that include EU customer rows, there's a risk these rows might end up on non-EU nodes.

Result Caching:

Definition: Many distributed query engines cache results to optimize performance and reduce computation time.
Risk: Cached results containing EU customer rows could be stored in non-EU locations or shared across different regions, violating data sovereignty laws.
Preventing Data Movement

Data Localization Strategy:

Ensure that all processing nodes are geographically close to the storage nodes within the EU. This can reduce the likelihood of intermediate shuffling.
Use a distributed file system like HDFS with strict geographic placement policies to store and access data only from EU-based nodes.

Shuffle Operations Control:

Implement custom shuffle mechanisms that ensure all intermediate data remains within the EU. For example, use local shuffles where possible or implement custom partitioning schemes.
Use Apache Spark’s spark.sql.shuffle.partitions configuration to control the number of partitions and optimize for minimal cross-border movement.

Result Caching Policies:

Configure caching mechanisms to only cache data within the EU. Implement strict policies that enforce data locality on cached results.
Use distributed caches like Hazelcast or Redis with regional availability zones to ensure that cached data is stored in the same region as the originating storage nodes.

Network Segmentation:

Segment your network into distinct regions, ensuring that EU traffic does not flow outside the European Union boundaries unless explicitly required by business logic.
Use Virtual Private Clouds (VPC) or equivalent services to isolate data processing and caching within the EU.

Data Lineage Tracking:

Maintain a detailed audit of data lineage to track where each row of EU customer data is processed and cached at any point in time.
Implement logging mechanisms that can trace back the flow of data and identify any potential violations.

#### Closing Summary

By carefully managing shuffle operations, result caching, network segmentation, and leveraging data localization strategies, we can ensure that EU customer rows remain physically within the European Union. This requires a combination of technical controls and organizational policies to enforce strict data sovereignty rules.

#### Common Mistakes
and Follow-Ups
Mistake 1: Failing to implement custom shuffle mechanisms leads to accidental movement of data across borders.
Mistake 2: Over-relying on default caching mechanisms without configuring them for regional availability can lead to cached results being stored outside the EU.

#### Follow-Up
Discussing how to handle edge cases like cross-region backups or disaster recovery plans that might inadvertently move data out of compliance.

**Comparison:**

```
Node	Region
node1	EU-West
node2	EU-West
node3	EU-Central
node4	EU-Central
Node	Region
node5	EU-West
node6	EU-West
node7	EU-Central
node8	EU-Central
```

**Code:**

```python
# Example configuration for Spark shuffle partitions and caching policies
spark.conf.set("spark.sql.shuffle.partitions", "50")  # Reduce number of shuffles
spark.conf.set("spark.sql.execution.arrow.inMemoryStorage.enabled", "false")  # Disable in-memory storage
```

**Example input / output:**

```text
Input: EU customer query with global result caching enabled
Output: Intermediate data stored in EU nodes, final results cached only within EU regions
```

**Data:**

```json
{
  "storageNodes": {
    "EU-West": ["node1-eu-west-1", "node2-eu-west-2"],
    "EU-Central": ["node3-eu-central-1", "node4-eu-central-2"]
  },
  "queryEngineNodes": {
    "EU-West": ["node5-eu-west-1", "node6-eu-west-2"],
    "EU-Central": ["node7-eu-central-1", "node8-eu-central-2"]
  }
}
```

**Flow:**

1. Query Initiation
2. Is Shuffle Required?
3. Partition and Local Shuffle
4. Remote Shuffle Across Borders
5. Cache Results Locally in EUNodes
6. Cache Results RemotelyOutside EU
7. Final Result within EU Storage
8. Final Result outside EU Storage

