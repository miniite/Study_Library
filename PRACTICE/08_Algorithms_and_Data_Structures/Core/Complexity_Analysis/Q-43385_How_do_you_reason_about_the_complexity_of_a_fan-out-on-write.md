---
type: "practice"
difficulty: "Expert"
---

# How do you reason about the complexity of a fan-out-on-write versus fan-out-on-read timeline/feed design, and how does follower-count skew change the calculus?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sorting Cost in Services](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Cost_in_Services.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In reasoning about the time and space complexities of a fan-out-on-write (FOOW) versus fan-out-on-read (FLOOR) timeline/feed design, we need to consider how operations are distributed across nodes, especially with follower-count skew. FOOW is generally more complex due to write amplification, whereas FLOOR can suffer from read latency issues under certain conditions.

#### Triaging the Question

The core of this question revolves around understanding and comparing the operational complexities of two different timeline/feed designs: fan-out-on-write (FOOW) and fan-out-on-read (FLOOR). The interviewer is probing your ability to analyze these systems, particularly focusing on how follower counts can significantly influence their performance characteristics.

#### Assumptions
Fan-Out-On-Write (FOOW): Data writes are distributed across multiple nodes.
Fan-Out-On-Read (FLOOR): Data reads are distributed across multiple nodes.
Follower Count Skew: Some nodes may have significantly more followers than others.

#### First-Principles Solution
Fan-Out-On-Write (FOOW):
Write Complexity:
Each write operation must be propagated to all follower nodes.
If there are ( n ) follower nodes, the write complexity is ( O(n) ).
Read Complexity:
Reads from a single node are straightforward and have a read complexity of ( O(1) ).
Follower Count Skew:
If some followers have significantly more users than others, writes to those nodes may become a bottleneck.
Space Complexity:
Each write requires storage on every follower node.
Fan-Out-On-Read (FLOOR):
Write Complexity:
Writes are centralized, typically at the master node or leader node.
The complexity of writing is ( O(1) ).
Read Complexity:
Reads can be distributed to multiple nodes, leading to a read complexity of ( O(\frac{n}{k}) ), where ( k ) is the number of nodes servicing reads.
Follower Count Skew:
Follower count skew doesn’t directly impact the write operation but can cause uneven load distribution among read nodes.
Space Complexity:
The master node holds all data, so space complexity is ( O(n) ), where ( n ) is the number of writes.

#### Closing Summary

FOOW and FLOOR have different strengths and weaknesses. FOOW ensures consistent low-latency reads but introduces challenges with write propagation and storage overhead. FLOOR simplifies write operations but may lead to uneven read distribution and higher latency under follower-count skew.

#### Common Mistakes/Follow-Ups
Mistake 1: Overlooking the implications of follower count skew in both designs.

#### Follow-Up
1: How would you implement a hybrid approach that balances writes and reads?

#### Follow-Up
2: What measures could be taken to mitigate write amplification in FOOW?

**Comparison:**

| Design       | Write Complexity | Read Complexity | Space Complexity | Follower Count Skew Impact |
|--------------|------------------|-----------------|------------------|----------------------------|
| FOOW         | O(n)             | O(1)            | O(n)             | Potential bottleneck       |
| FLOOR        | O(1)             | O(n/k)          | O(n)             | Uneven read load           |

**Code:**

```python
# Example code for simulating write propagation in FOOW
def foow_write(node_data, follower_nodes):
    for node in follower_nodes:
        node_data[node] = "updated"

# Example code for reading from a single node in FLOOR
def floor_read(master_node):
    return master_node.get("data")
```

**Example input / output:**

```

FOOW Write Example Input/Output:

PLAINTEXT
Copy
Input:
node_data = {"follower1": "initial", "follower2": "initial"}
follower_nodes = ["follower1", "follower2"]

Output:
node_data = {"follower1": "updated", "follower2": "updated"}

FLOOR Read Example Input/Output:

PLAINTEXT
Copy
Input:
master_node = {"data": "centralized_data"}

Output:
data = "centralized_data"
```

**Data:**

```json
{
  "node_data": {
    "follower1": "initial",
    "follower2": "initial"
  },
  "master_node": {
    "data": "centralized_data"
  }
}
```

**Flow:**

1. Yes
2. No
3. Write Operation
4. Follower Count Skew?
5. Centralized Write + DistributedRead
6. Distributed Writes +Centralized Reads
7. Floor Read Latency Issues
8. Fan-Out-On-Write Bottleneck

