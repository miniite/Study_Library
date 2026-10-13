---
type: "practice"
difficulty: "Deep"
---

# When does Photon actually help and when does it give you nothing, and how do you measure the DBU tradeoff of enabling it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Databricks Ingestion](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Ingestion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Photon in Databricks is a hardware-accelerated engine designed to speed up SQL queries by leveraging GPU power. When deciding whether Photon helps or not, and how to measure the DBU tradeoff of enabling it, we need to consider several factors.

#### Triaging the question
The core issue here is understanding when Photon's benefits outweigh its cost. This involves measuring query performance with and without Photon, assessing hardware utilization, and evaluating the overall impact on system resources.

#### First-principles solution

Query Characteristics: Photon is most effective for complex queries involving vectorized operations, such as those involving machine learning models or large matrix computations.
Data Size and Complexity: Larger datasets and more complex queries benefit more from Photon's acceleration capabilities.
Hardware Utilization: Measuring the GPU utilization before and after enabling Photon can provide insights into its effectiveness.
Cost-Benefit Analysis: Comparing the DBUs consumed by Photon-enabled queries with those of non-Photon queries helps in determining if the added cost is justified.
Step-by-Step Solution

Identify Suitable Queries:

Complexity Check: Use Databricks' query profiling tools to identify complex, vectorized operations.
Sample Queries: Start by running a few representative queries that involve heavy computation or large datasets.

Enable Photon Temporarily:

Modify the cluster configuration to enable Photon for specific queries.
Run these queries and record their execution times and resource utilization.

Measure Performance Impact:

Compare the performance metrics (e.g., query runtime, memory usage) before and after enabling Photon.
Use tools like Databricks SQL Performance Analytics or third-party monitoring tools to gather detailed performance data.

Evaluate GPU Utilization:

Monitor GPU utilization during query execution using Databricks' built-in monitoring capabilities.
Analyze the impact of Photon on overall cluster resource allocation and availability.

DBU Tradeoff Analysis:

Calculate the DBUs consumed by queries with and without Photon.
Determine if the performance gains justify the additional cost incurred by DBUs.

Iterate and Refine:

Continuously refine the queries and configurations to optimize performance further.
Ensure that the use of Photon does not negatively impact other query workloads or resource constraints.

#### Closing Summary

Photon can significantly enhance query performance for complex, vectorized operations but comes with an associated cost. To measure its effectiveness, one must carefully select representative queries, monitor performance metrics, and evaluate the tradeoff between improved query times and increased DBUs. This approach ensures that Photon is used optimally without overspending.

#### Common Mistakes/Follow-Ups
Overusing Photon: Running all queries with Photon can lead to suboptimal resource utilization.
Neglecting Cost: Failing to monitor and account for the additional DBU cost can result in unnecessary expenses.
Misalignment of Workloads: Ensuring that Photon is enabled only on workloads where it provides significant benefits.

**Comparison:**

```
Metric	Without Photon	With Photon
GPU Utilization (%)	10	80
DBUs Consumed (per query)	2.5	4.0
Input-Output Example
PLAINTEXT
Copy
Input:
- Run a complex query on a large dataset using Databricks.

Output:
- GPU Utilization: 10% without Photon, 80% with Photon.
- DBUs Consumed: 2.5 DBUs per query without Photon, 4.0 DBUs per query with Photon.

Conclusion:
- Photon led to significant GPU utilization and higher DBU consumption.
```

**Code:**

```python
# Example Python code to enable Photon and run a query
from databricks import sql

query = "SELECT * FROM large_table"
with sql.connect(server_hostname="your-server", http_path="your-http-path") as connection:
    cursor = connection.cursor()
    cursor.execute(query)
    result = cursor.fetchall()

# Measure execution time
import time
start_time = time.time()
result = cursor.fetchall()
end_time = time.time()
print(f"Execution Time: {end_time - start_time} seconds")
```

**Example input / output:**

```
Query Type	Execution Time Without Photon (s)	Execution Time With Photon (s)
ML model training	120.5	60.2
Input-Output Example
PLAINTEXT
Copy
Input:
- Run a complex query on a large dataset using Databricks.

Output:
- Execution time: 120.5 seconds (without Photon)
- Execution time: 60.2 seconds (with Photon)

Conclusion:
- Photon reduced the execution time by approximately 50% for this specific query.
```

**Data:**

```json
{
  "query_id": 12345,
  "cluster_id": "c-101",
  "query_type": "ML model training",
  "data_size": "1GB",
  "execution_time_without_photon": 120.5,
  "execution_time_with_photon": 60.2
}
```

**Flow:**

1. Yes
2. Identify Suitable Queries
3. Complexity Check
4. Enable Photon Temporarily
5. RUN & MONITOR queries
6. Measure Performance Impact
7. Evaluate GPU Utilization
8. DBU Tradeoff Analysis
9. Iterate and Refine

