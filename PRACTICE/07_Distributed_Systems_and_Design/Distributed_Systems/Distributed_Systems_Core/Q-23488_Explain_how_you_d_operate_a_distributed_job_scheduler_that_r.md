---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd operate a distributed job scheduler that runs each job exactly once at its scheduled time across a fleet, tolerating node failures without duplicates or misses. What leader election and sharding do you use?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Leader_Election.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our distributed job scheduler operates flawlessly—delivering jobs exactly once at their scheduled times while tolerating node failures and avoiding duplicates or misses—we need to design a robust system that involves leader election, sharding, and distributed consensus mechanisms. The core idea is to partition the workload efficiently across nodes and ensure reliable communication and state management.

#### Triaging the Question

The question revolves around designing a resilient job scheduler in a distributed system environment. Specifically, it requires addressing:

Leader Election: Ensuring there's one central node (leader) that coordinates task scheduling.
Sharding: Distributing jobs across multiple nodes to balance load and improve scalability.

#### Cons
istency: Avoiding duplicates or missed tasks due to node failures.

#### First-Principles Solution
Leader Election:
We can use a consensus algorithm like Raft or Paxos to elect a leader among the nodes. The leader is responsible for receiving job requests and scheduling them.
Sharding:
Jobs are sharded based on some key, such as a hash of their scheduled time or unique ID, to distribute evenly across nodes.
Reliable Communication:
Nodes communicate using a reliable messaging system (e.g., Kafka) for job updates and status reports.
Detailed Solution

Leader Election with Raft:

Initialization: Each node starts in the "Follower" state, awaiting leadership.
Election Process: When no leader is available or a node detects one has failed, it transitions to the "Candidate" state and requests votes from other nodes.

#### Cons
ensus: The majority of nodes voting for a candidate makes them the new leader. If not elected, they transition back to the "Follower" state.

Sharding Jobs:

Each job is assigned a unique ID or scheduled time. We can hash this value to determine which node handles it.
For example, using hash(job_id) % num_nodes to distribute jobs across nodes.

Distributed Job Scheduling:

The leader schedules each job by sending it to the appropriate shard (node).
Nodes acknowledge receipt and execute the job once it's their turn.
Upon completion or failure, nodes report back to the leader for reconciliation.

Handling Failures:

If a node fails, other nodes detect this through periodic heartbeats and redistribute tasks as needed.
The leader can be elected from among surviving nodes if necessary.

Avoiding Duplicates/Misses:

Each job is tagged with metadata ensuring its uniqueness (e.g., timestamp + unique identifier).
Nodes check the job’s metadata before execution to avoid reprocessing duplicates or missing tasks.

#### Closing Summary

By implementing a leader election mechanism using Raft, sharding jobs based on their unique identifiers, and ensuring reliable communication, we can achieve a highly available and fault-tolerant distributed job scheduler. This approach ensures that each job runs exactly once at its scheduled time without being duplicated or missed due to node failures.

#### Common Mistakes/Follow-Ups

Mistakes:

Ignoring the need for consensus algorithms like Raft, leading to unreliable leadership.
Not properly sharding jobs can result in uneven load distribution and suboptimal performance.
Inadequate failure handling mechanisms can lead to data inconsistencies or task duplication.

#### Follow-Ups

What messaging protocols would you use for reliable job execution?
How would you handle edge cases like partial node failures during leader elections?
Are there any trade-offs in choosing Raft over Paxos?

**Comparison:**

```
Node ID	State	Heartbeat Status
0	Follower	Active
1	Leader	Active
2	Follower	Inactive
Job ID	Scheduled Time	Node Assigned
1	1584639200	0
2	1584640000	1
3	1584640800	2
```

**Code:**

```python
# Example Raft Leader Election Code (simplified)
class Node:
    def __init__(self, id):
        self.id = id
        self.state = "follower"

def elect_leader(nodes):
    for node_id in nodes:
        if nodes[node_id].state == "candidate":
            # Election process logic here
            pass

# Example Job Scheduling Code (simplified)
class JobScheduler:
    def __init__(self, num_nodes):
        self.nodes = [Node(i) for i in range(num_nodes)]

    def schedule_job(self, job_id):
        node_index = hash(job_id) % len(self.nodes)
        self.nodes[node_index].execute_job(job_id)
```

**Example input / output:**

```text
Input:
- Job Scheduling Request: Schedule job_id = 2 at time = 1584640000

Output:
- Leader Node (e.g., node with id=1) receives the request.
- Leader schedules the job on node with id=(hash(2) % 3) = 1
- Node 1 acknowledges and executes the job.
- Job is completed, status reported back to leader for reconciliation.
```

**Data:**

```json
{
  "nodes": [
    { "id": 0 },
    { "id": 1 },
    { "id": 2 }
  ],
  "jobs": [
    { "id": 1, "scheduled_time": 1584639200 },
    { "id": 2, "scheduled_time": 1584640000 },
    { "id": 3, "scheduled_time": 1584640800 }
  ]
}
```

**Flow:**

1. Heartbeat
2. Votes
3. Election Timeout
4. Timeout
5. (Follower State)
6. (Candidate State)
7. (Leader State)
8. (Job Request)
9. (Raft Leader)
10. (Node Execution)
11. StatusReport

