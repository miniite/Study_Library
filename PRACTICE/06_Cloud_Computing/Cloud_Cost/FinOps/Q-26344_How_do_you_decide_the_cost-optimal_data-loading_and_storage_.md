---
type: "practice"
difficulty: "Expert"
---

# How do you decide the cost-optimal data-loading and storage architecture for training (local NVMe cache vs object store vs parallel filesystem like Lustre/FSx), given that a data-starved GPU pipeline wastes the most expensive resource you own? Walk me through when the storage premium pays for itself in GPU utilization.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Parallelism Strategies](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_of_Parallelism_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide the cost-optimal data-loading and storage architecture for training, we need to balance the cost of different storage solutions with their performance impact on GPU utilization. Given that a data-starved pipeline is the most costly resource, our goal is to ensure that the chosen storage solution maximizes GPU efficiency.

The key factors include:

Local NVMe Cache: Fast access but limited in capacity and cost.
Object Store: Scalable and cheap, but slower access times due to network latency.
Parallel Filesystems (Lustre/FSx): Balancing between cost and performance with distributed access.

Given the high value of GPU time, we need a solution that can provide quick data access without excessive costs. Let’s walk through the decision-making process from first principles.

#### Triage

The question is asking us to choose the optimal storage architecture by evaluating the trade-offs between cost and performance. We will assume:

GPUs are the most valuable resource.
Cost optimization is crucial, but not at the expense of training time.

#### First Principles Solution
1. Local NVMe Cache
Advantages: Fast data access, no network latency.
Disadvantages: Limited storage capacity and cost.
Cost Optimal for Small Datasets: Suitable when datasets fit within local storage limits (e.g., a few GBs to tens of TBs).
2. Object Store
Advantages: Scalable, cheap per GB stored.
Disadvantages: Slower access due to network latency.
Cost Effective for Large Datasets: Ideal when datasets are too large for local storage.
3. Parallel Filesystems (Lustre/FSx)
Advantages: Distributed, high-performance access with scalable storage.
Disadvantages: Costlier than object stores but cheaper than NVMe.
Balanced Performance and Cost: Suitable when you need both performance and cost efficiency.
Detailed Analysis
Local NVMe Cache

For small datasets (e.g., 1-50 TB), local NVMe storage is cost-effective. It provides fast access times, minimizing the risk of data-starving GPUs. However, as dataset sizes increase, this approach becomes less feasible due to capacity limitations and costs.

Small Datasets (1-50TB)
Local NVMe Cache
Object Store or Lustre/FSx
Object Store

For very large datasets, the object store becomes more attractive due to its scalability and cost-effectiveness. The trade-off is network latency, which can be mitigated by careful network design and caching strategies.

Large Datasets (>50TB)
Object Store
Local NVMe Cache orLustre/FSx with NetworkOptimizations
Parallel Filesystems (Lustre/FSx)

For large datasets and high performance, parallel file systems offer a balanced approach. They can provide the necessary performance while keeping costs lower than local NVMe solutions.

Large Datasets (>50TB)
Lustre/FSx
Local NVMe Cache (for criticaldatasets)

#### Closing Summary
and Common Mistakes

The optimal choice depends on the dataset size. For small to medium-sized datasets, local NVMe cache is cost-effective due to its fast access times. As the dataset grows, parallel file systems offer a balanced solution between performance and cost. Object stores are suitable for extremely large datasets where cost constraints are more critical.

#### Common mistakes
include:

Overlooking network latency when using object stores.
Ignoring the capacity limits of local NVMe solutions.
Failing to leverage distributed storage strategies effectively in parallel filesystems.
Final Considerations
Network Optimization: Ensure robust networking between GPUs and storage for object stores.
Hybrid Solutions: Combining multiple storage solutions (e.g., using an object store as a back-end and a local cache as a front-end) can optimize performance and cost further.
Automation: Implementing automated data tiering or caching strategies can dynamically switch datasets to the most appropriate storage solution.

**Comparison:**

```
Dataset Size	Local NVMe Cache	Object Store	Parallel Filesystem
1-50GB	Cost-effective	Not feasible	Feasible
>50-1000GB	Feasible	Cost-effective	Balanced
>1000GB	Not feasible	Cost-effective	Cost-effective
```

**Code:**

```python
# Example Python script for automated data tiering
def select_storage(dataset_size):
    if dataset_size <= 50:
        return "local_nvme"
    elif dataset_size > 1000:
        return "object_store"
    else:
        return "parallel_filesystem"
```

**Example input / output:**

```text
Input:
- Dataset size: 30GB

Output:
- Storage Solution: local_nvme
Explanation: The dataset fits within the capacity of a local NVMe cache, providing fast access times.
```

**Data:**

```json
{
  "dataset_sizes": {
    "small": [1, 20],
    "medium": [51, 499],
    "large": [500, 1000]
  },
  "storage_options": {
    "local_nvme": {"cost_per_gb": 0.1, "access_time_ms": 0},
    "object_store": {"cost_per_gb": 0.01, "access_time_ms": 500},
    "parallel_filesystem": {"cost_per_gb": 0.02, "access_time_ms": 100}
  }
}
```

**Flow:**

1. Dataset Size <= 50GB
2. Local NVMe Cache
3. Object Store or ParallelFilesystem
4. Large (>1000GB)
5. Parallel Filesystem
6. Medium (51-499GB)
7. Parallel Filesystem

