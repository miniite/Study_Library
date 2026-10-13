---
type: "practice"
difficulty: "Deep"
---

# A principal on another team ships a design that puts a monotonic timestamp partition key on a global write-heavy table. In the design review, how do you explain the multi-region hot-shard failure it will cause at scale and steer them without undermining their authority?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Wide Column Design and Migration](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Wide_Column_Design_and_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue with the monotonic timestamp partition key in a global write-heavy table design, we need to understand the underlying storage patterns of Cassandra/DynamoDB and the implications of hot-shards across multiple regions. A well-structured explanation can steer the team towards better design choices without undermining their authority.

#### Triaging the Question

The core issue is that using a monotonic timestamp as the partition key in a highly write-intensive table will likely lead to hot-shards, particularly in a multi-region setup. The key points we need to cover are:

Partition Key Selection: How it impacts data distribution and read/write performance.
Hot-Shard Effect: What happens when too many writes converge on the same partition in different regions.
Scalability Considerations: Why this can lead to uneven load across nodes, impacting overall system availability.

#### First-Principles Solution

Partition Key and Data Distribution:

In Cassandra/DynamoDB, the partition key is used to determine which node stores a particular piece of data.
A monotonic timestamp (e.g., current time in milliseconds) will cause writes to converge on the same partition as the timestamp values are likely to be similar across different write operations.

Hot-Shard Failure:

In a multi-region setup, if the writes from all regions are converging on the same partition due to the monotonic timestamp, this can lead to uneven load distribution.
This results in hot-shards, where one or more partitions handle an disproportionate amount of read/write traffic, leading to performance degradation and potential single points of failure.

Steering Without Undermining Authority:

Acknowledge their expertise while providing data-driven insights.
Highlight the importance of understanding the storage patterns and their impact on scalability and availability.
Suggest alternative partition keys that can better distribute writes across partitions, ensuring more even load distribution.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Using a monotonic timestamp as a partition key in a global write-heavy table will likely result in hot-shards, causing uneven load distribution and potential performance issues. It's crucial to choose a partition key that distributes writes evenly across partitions.

#### Common Mistakes
Overlooking the impact of partition keys on data distribution.
Assuming all timestamps are sufficiently unique without considering concurrent write operations.
Triaging Continued

To further support this, we can present an example:

#### Example Scenario

#### Cons
ider a global table with a monotonic timestamp as the partition key. In one hour, there could be 100 writes per second from each of 3 regions (A, B, and C). If the timestamps are clustered around the same time interval, all these writes might converge on a single partition in each region.

Input-Output Example
CODE
Input: Region A - timestamp 1628400000000, write operation.
       Region B - timestamp 1628400000000, write operation.
       Region C - timestamp 1628400000000, write operation.

Output: In each region, a single partition handles the writes from all three regions. This leads to hot-shards in each region.

#### Conclusion

By explaining these points clearly and providing an example, we can steer the team towards better design choices without undermining their authority. The key is to align with their expertise while guiding them toward more scalable solutions.

**Comparison:**

```
Region	Timestamp (ms)	Write Operation
A	1628400000000	write1
B	1628400000000	write3
C	1628400000000	write5
```

**Code:**

```python
# Example Python code snippet for generating timestamp-based partition keys
import time

def generate_partition_key():
    return int(time.time() * 1000)
```

**Example input / output:**

```text
Input:
Region A - timestamp 1628400000000, write operation.
Region B - timestamp 1628400000000, write operation.
Region C - timestamp 1628400000000, write operation.

Output:
In each region, a single partition handles the writes from all three regions. This leads to hot-shards in each region.
```

**Data:**

```json
{
  "regionA_writes": [
    {"timestamp": 1628400000000, "data": "write1"},
    {"timestamp": 1628400000500, "data": "write2"}
  ],
  "regionB_writes": [
    {"timestamp": 1628400000000, "data": "write3"},
    {"timestamp": 1628400000500, "data": "write4"}
  ],
  "regionC_writes": [
    {"timestamp": 1628400000000, "data": "write5"},
    {"timestamp": 1628400000500, "data": "write6"}
  ]
}
```

**Flow:**

1. Region A
2. Partition
3. Region B
4. Region C
5. Hot-Shard

