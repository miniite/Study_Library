---
type: "practice"
difficulty: "Deep"
---

# Spanner charges for compute in processing units/nodes and storage separately; model the cost of a workload doing 50k QPS multi-region reads and 5k QPS writes, and how you would optimize spend without losing consistency.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Query Execution](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Distributed_Query_Execution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When modeling costs in Google Cloud Spanner, it's crucial to understand that Spanner charges separately for storage and compute resources. In this scenario, we are tasked with modeling a workload involving 50k QPS multi-region reads and 5k QPS writes, and then exploring optimization strategies without compromising consistency.

Firstly, let’s break down the costs:

Storage Costs: These depend on the amount of data stored.
Compute Costs (Processing Units/Nodes): This includes both read and write operations, as Spanner charges for each operation based on its complexity and duration.

To model the cost accurately, we need to know the specific details such as the size of data per row, number of rows involved in reads/writes, and the distribution of the workload across regions. For this example, let’s assume the following:

Each row is 1KB.
There are 50 million rows.
The read/write ratio remains constant.

The cost model can be represented as follows:

[ \text{Total Cost} = \text{Storage Cost} + (\text{Reads QPS} \times \text{Cost per Read}) + (\text{Writes QPS} \times \text{Cost per Write}) ]

Assuming the costs are:

$0.25 per GB of storage.
$0.18 per 1,000 reads (at most one processing unit/node).
$0.36 per 1,000 writes (at least two processing units/nodes).

Now, let's calculate the costs:

Storage Cost: [ \text{Total Storage} = 50,000,000 \times 1KB = 50GB ] [ \text{Storage Cost} = 50 \times 0.25 = $12.50 ]

Reads QPS: [ \text{Total Reads QPS} = 50,000 ] [ \text{Cost per Read} = 0.18 ] [ \text{Read Cost} = \frac{50,000}{1,000} \times 0.18 = $9 ]

Writes QPS: [ \text{Total Writes QPS} = 5,000 ] [ \text{Cost per Write} = 0.36 ] [ \text{Write Cost} = \frac{5,000}{1,000} \times 0.36 = $1.80 ]

Adding these costs together: [ \text{Total Cost} = 12.50 + 9 + 1.80 = $23.30 ]

To optimize the spend without losing consistency, we can consider several strategies:

Optimization Strategies

Sharding and Partitioning: Spanner supports sharding and partitioning of tables to reduce contention and improve read performance. By distributing rows across multiple partitions or shards, you can distribute the load more evenly.

Use Materialized Views: For frequently accessed data, materializing views can help reduce the number of reads required for complex queries.

Reduce Redundant Reads: Ensure that your query patterns are optimized to avoid unnecessary reads. For example, use appropriate indexing and partitioning strategies to minimize the read volume.

Upgrade Hardware: If you find that your current configuration is not sufficient, consider upgrading to a higher tier of Spanner which offers more processing units/nodes.

Distribute Reads Across Regions: Since this workload includes multi-region reads, ensure that data is evenly distributed across regions to leverage the proximity advantage and reduce latency.

Use Transactional Control: Fine-tune your transactional control to avoid unnecessary consistency levels. Spanner allows you to set different consistency models (eventual, read-your-writes) based on your application needs, which can help in reducing costs.

Monitor and Tune: Regularly monitor the performance metrics of your Spanner deployment and tune as needed. Tools like Cloud Monitoring and Cloud Profiler can provide insights into resource usage and bottlenecks.

#### Closing Summary

In summary, modeling the cost involves understanding the specific workload characteristics and then applying various optimization strategies to reduce costs while maintaining consistency. Sharding, materialized views, and fine-grained transactional control are key techniques for achieving this balance.

#### Common Mistakes

Overlooking the need for consistent data distribution across regions.
Not leveraging materialized views to optimize read-heavy workloads.
Failing to monitor performance and make timely adjustments based on real-world usage patterns.

**Comparison:**

```
Operation	QPS	Cost per 1,000	Cost
Storage	-	-	$12.50
Reads	50k	$0.18	$9.00
Writes	5k	$0.36	$1.80
Total	-		$23.30
```

**Code:**

```python
# Example Python code for cost calculation (hypothetical)
storage_cost = 50 * 0.25  # 50GB at $0.25 per GB
read_cost = (50000 / 1000) * 0.18  # 50k reads at $0.18 per 1k reads
write_cost = (5000 / 1000) * 0.36  # 5k writes at $0.36 per 1k writes

total_cost = storage_cost + read_cost + write_cost
print(f"Total Cost: ${total_cost:.2f}")
```

**Example input / output:**

```text
Input:
- 50M rows with 1KB per row
- 50k QPS reads and 5k QPS writes

Output:
Storage Cost: $12.50
Read Cost: $9.00
Write Cost: $1.80
Total Cost: $23.30
```

**Data:**

```yaml
# Example Spanner Schema
spanner_schema:
  - name: users
    columns:
      - {name: id, type: INT64}
      - {name: email, type: STRING(100)}
      - {name: last_updated, type: TIMESTAMP}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Start
6. Is workload distributionuniform?
7. Consider sharding andpartitioning
8. Are materialized views used forreads?
9. Create materialized views
10. Monitor performance and tuneas needed
11. Optimize hardwareconfiguration
12. Reduce redundant reads byindexing
13. Distribute reads across regions
14. Determine optimal consistencylevel
15. End

