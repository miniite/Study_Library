---
type: "practice"
difficulty: "Deep"
---

# A collaborative-filtering recommender builds a user-item interaction graph per request from normalized rows and it's slow; how do you decide between a precomputed materialized adjacency structure and building on the fly given your read/write ratio?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Set Intersection Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Set_Intersection_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the performance bottleneck in our collaborative-filtering recommender system, we need to balance the trade-offs between precomputing an adjacency structure and dynamically building it on-the-fly based on read/write ratios. The core decision involves understanding the nature of these operations and their impacts on both latency and resource utilization.

#### Triage
Understanding the Question

The question probes our ability to evaluate and optimize system performance by considering different data structures and algorithms, specifically focusing on the balance between read and write operations in a collaborative-filtering scenario. We need to assess whether precomputing an adjacency structure (i.e., storing user-item interactions in memory) can reduce latency at the cost of increased initial storage requirements and potential maintenance overhead.

#### Assumptions
Read/Write Ratio: There is a significant difference between read and write operations.
Latency Sensitivity: The application prioritizes fast response times for read operations over frequent updates (writes).
Resource Availability: We have limited memory but ample computational power to perform on-the-fly calculations.

#### First-Principles Solution
Step 1: Define the Problem

In a collaborative-filtering system, we need to quickly retrieve similar users or items based on interaction history. This involves querying an adjacency structure that maps user-item interactions.

Step 2: Evaluate Read-Write Operations
Read Operation: Fetching precomputed similarity scores or recommendations.
Write Operation: Updating the graph with new interactions or changes in preferences.

Given a high read/write ratio favoring reads, we can infer that the system spends most of its time querying rather than updating. This scenario makes precomputing more beneficial because it reduces the computational cost of each query.

Step 3: Analyze Memory vs. Computation Trade-offs
Precomputed Adjacency Structure: Requires upfront storage but significantly reduces per-query computation.
On-the-Fly Construction: Reduces memory footprint at the expense of increasing compute time during read operations.
Step 4: Cost-Benefit Analysis
Latency Impact:
Precomputation can drastically reduce query latency, ensuring faster responses for users.
Memory Usage:
If memory is a constraint, precomputing might not be feasible due to the large size of adjacency matrices in many real-world scenarios.
Maintenance Overhead:
Frequent updates may necessitate periodic recomputations or incremental updates to maintain accuracy.
Step 5: Implement Solution
Optimize for Reads: Use a precomputed structure like an adjacency matrix, where each entry represents the strength of the relationship between users/items.
Lazy Updates: For infrequent changes, use lazy updates where interactions are stored and batch-computed periodically or incrementally during high-demand times.

#### Closing Summary

In conclusion, we should opt for precomputing an adjacency structure when read operations significantly outnumber write operations. This approach minimizes query latency at the cost of initial memory usage. However, for scenarios with limited resources or frequent updates, on-the-fly construction can be a viable alternative by balancing memory and compute trade-offs.

#### Common Mistakes/Follow-Ups
Ignoring Write Operations: Overlooking potential updates to the graph during the decision-making process.
Memory Limitations: Assuming unlimited memory availability without considering real-world constraints.
Not Considering Incremental Updates: Failing to explore methods that balance between precomputation and on-the-fly construction, such as lazy updates.

**Code:**

```python
# Example of a simple adjacency matrix for precomputed data
adj_matrix = {
    'user1': {'item1': 0.85, 'item2': 0.63},
    'user2': {'item1': 0.78, 'item3': 0.92}
}

# Lazy update example (simplified)
def lazy_update(user: str, item: str, rating: float):
    # Update the matrix in a batch during high-demand times
    adj_matrix.setdefault(user, {})[item] = rating
```

**Example input / output:**

| User  | Item   | Rating |
|-------|--------|--------|
| user1 | item1  | 0.85   |
| user2 | item1  | 0.78   |
| user1 | item2  | 0.63   |
| user2 | item3  | 0.92   |

**Data:**

```json
{
  "users": ["user1", "user2", "user3"],
  "items": ["item1", "item2", "item3"]
}
```

