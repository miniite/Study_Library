---
type: "practice"
difficulty: "Deep"
---

# Your Redis is single-threaded and one slow O(n) command (KEYS, big SMEMBERS) stalls everyone. How do you find the offending command, and what alternatives (SCAN, keyspace design) fix it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency After Expansion](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a single-threaded Redis instance being stalled by an O(n) command like KEYS or big SMEMBERS, we need to identify the offending command, understand its impact on performance, and then explore alternatives such as using SCAN, optimizing keyspace design, and leveraging Redis clusters. The key challenge here is to ensure that our system remains responsive even under slow operations.

The question primarily tests your ability to diagnose issues in a single-threaded environment, understand the trade-offs between different strategies, and design scalable solutions. Here are the steps we'll follow:

Identify the offending command.
Understand its impact on performance.
Explore alternatives like SCAN.

#### Cons
ider keyspace design optimizations.
Evaluate Redis cluster as an alternative.

Let's dive into each step in detail.

**Comparison:**

```
Command	Impact
KEYS	Blocks all other commands until completion
SMEMBERS	Similar impact as KEYS if the set is large
```

**Code:**

```python
# Example function to simulate a slow O(n) command
def slow_command(keys):
    import time
    time.sleep(10)  # Simulating a long running operation
    return keys
```

**Example input / output:**

```

For the slow_command function with input:

PYTHON
Copy
slow_command(["key1", "key2", "key3"])

Output would be a delayed response due to the sleep operation, simulating the slow command.
```

**Data:**

```json
{
  "keys": ["key1", "key2", "key3", ...] // A large list of keys
}
```

**Flow:**

1. Yes
2. No
3. Identify Offending Command
4. Is it a slow operation?
5. Use SCAN to paginate results
6. Consider keyspaceoptimization or cluster setup
7. Implement pagination logicwith SCAN
8. Optimize keyspace design forsmaller chunks
9. Set up Redis Cluster forhorizontal scalability

