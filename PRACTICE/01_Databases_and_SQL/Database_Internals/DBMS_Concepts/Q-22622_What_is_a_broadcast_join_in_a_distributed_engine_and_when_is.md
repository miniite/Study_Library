---
type: "practice"
difficulty: "Deep"
---

# What is a broadcast join in a distributed engine and when is it preferred over a shuffle (hash) join?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Partitioning and Partition Pruning](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Broadcast joins are a type of join operation used in distributed databases where one of the smaller tables is entirely loaded into memory on each node, and then that data is broadcast to every other node in the cluster during the execution of a query. This approach contrasts with shuffle (hash) joins, which rely on redistributing data across nodes using hash functions.

#### Triaging the Question

The question asks us to explain the concept of broadcast joins within distributed database engines and to understand when it would be preferred over shuffle (hash) joins. The interviewer is likely probing your understanding of join operations in distributed systems, the trade-offs between these two methods, and your ability to reason about data distribution strategies.

Core Concept

Broadcast joins are favored for small datasets that can fit into memory because they reduce the amount of network traffic compared to shuffle joins. In contrast, shuffle joins use a hash-based partitioning method that requires more communication and coordination across nodes.

#### Assumptions
Both broadcast join and shuffle join operate in distributed environments.
The smaller table is known to be significantly smaller than the larger one.
Memory constraints are manageable for small tables but not for large ones.

#### First-Principles Solution

Broadcast Join:

How it works: One of the smaller tables (the “small” or “broadcast” table) is loaded entirely into memory on each node in the cluster, and then broadcast to every other node.
Advantages:
Reduces network traffic since the smaller table doesn’t need to be shuffled around.
Can significantly speed up queries if the small table is relatively small compared to the large one.
Disadvantages:
Requires enough memory on each node for the small table, which can limit scalability.

Shuffle Join (Hash Join):

How it works: Both tables are partitioned based on a common key using hash functions, and then joined by matching partitions.
Advantages:
Scalable across multiple nodes as data is distributed evenly.
Flexible in handling large datasets since the small table can be broadcast to all nodes before joining.
Disadvantages:
Requires significant network bandwidth due to shuffling and rebalancing of partitions.
When to Use Broadcast Joins

Broadcast joins are preferred over shuffle joins when:

The smaller table is very small, fitting comfortably into memory on each node.
Reducing network traffic is critical for performance.
You have a high cardinality key that can effectively partition the larger table but not enough nodes or memory capacity.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, broadcast joins are ideal for scenarios where the smaller dataset fits into memory, reducing unnecessary shuffling. However, they come with the downside of requiring sufficient memory on each node. On the other hand, shuffle joins offer better scalability but at a cost of increased network traffic.

#### Common Mistakes
Misunderstanding the memory constraints and failing to consider whether the small table can indeed fit into memory.
Overlooking that broadcast joins are only beneficial when applied to significantly smaller datasets compared to shuffle joins.

#### Follow-ups
might include discussing how data size, memory limitations, and network bandwidth impact performance decisions or exploring alternative join methods like sort merge joins in distributed environments.

**Comparison:**

| small_table | large_table |
|-------------|-------------|
| id          | key         | id        | key     | value   |
| 1           | A           | 101       | A       | X       |
| 2           | B           | 102       | C       | Y       |
|             |             | 103       | B       | Z       |

**Code:**

```python
# Example code for a simple broadcast join (pseudo-code)
def broadcast_join(smaller_table, larger_table):
    # Load smaller table into memory on each node
    broadcast_small_table = load_into_memory(smaller_table)

    # Perform the join by broadcasting small table to all nodes and joining with large table
    result = []
    for node in nodes:
        broadcast(broadcast_small_table)  # Broadcast small table to node
        local_result = node.join(broadcast_small_table, larger_table)
        result.extend(local_result)

    return result
```

**Example input / output:**

```text
Input:
small_table = [{"id": 1, "key": "A"}, {"id": 2, "key": "B"}]
large_table = [{"id": 101, "key": "A", "value": "X"}, {"id": 102, "key": "C", "value": "Y"}, {"id": 103, "key": "B", "value": "Z"}]

Output:
[
    {"small_id": 1, "large_id": 101, "key": "A", "value": "X"},
    {"small_id": 2, "large_id": 103, "key": "B", "value": "Z"}
]
```

**Data:**

```json
{
  "small_table": [
    {"id": 1, "key": "A"},
    {"id": 2, "key": "B"}
  ],
  "large_table": [
    {"id": 101, "key": "A", "value": "X"},
    {"id": 102, "key": "C", "value": "Y"},
    {"id": 103, "key": "B", "value": "Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is small table < memory limit?
5. Load into memory andbroadcast
6. Shuffle large table, performjoin
7. Perform join locally
8. Join with large table ondistributed nodes
9. Return result

