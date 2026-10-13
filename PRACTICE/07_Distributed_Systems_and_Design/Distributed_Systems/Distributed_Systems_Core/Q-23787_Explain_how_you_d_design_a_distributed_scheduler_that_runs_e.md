---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a distributed scheduler that runs each team's scheduled retraining exactly once at its scheduled time across a fleet, tolerating node failure, and prevents a thundering herd of retrains all firing at midnight from swamping the shared GPU cluster.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Locks and Fencing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_Locks_and_Fencing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed scheduler that runs each team's scheduled retraining exactly once at its scheduled time across a fleet, while tolerating node failures and preventing a thundering herd of retrains from swamping the shared GPU cluster, we need to focus on ensuring reliable scheduling, fault tolerance, and efficient resource management. The key concepts involve using a consensus protocol for distributed coordination, a robust task queue with retries, and smart load balancing.

Firstly, let's break down what the system needs to achieve:

Reliable Scheduling: Each retraining job must run exactly once at its scheduled time.
Fault Tolerance: The scheduler should handle node failures gracefully without losing jobs or causing delays in execution.
Throttling Mechanism: To prevent a large number of retrains from firing at the same time, such as midnight, which could overwhelm the GPU cluster.

#### Triaging the Question

The interviewer is probing our understanding of distributed systems concepts like consensus algorithms (e.g., Raft or Paxos), task scheduling strategies, and load balancing techniques. We need to assume a basic architecture with nodes that can communicate with each other using network protocols, and we will consider common failure modes such as node crashes.

#### First-Principles Solution

#### Cons
ensus Protocol: Use a consensus protocol like Raft or Paxos to ensure that the scheduler agrees on which nodes should execute tasks at specific times.
Task Queue with Retries: Design a task queue where each job is retried automatically if it fails, ensuring that no job is lost due to temporary failures.
Load Balancing and Throttling: Implement mechanisms to balance the load across nodes and prevent thundering herds by spreading out the execution of tasks.
Step-by-Step Solution

#### Cons
ensus Protocol (e.g., Raft)

Establish a leader-follower model where one node is elected as the leader.
The leader schedules jobs for specific nodes based on their availability and the current time.
Nodes run tasks only if they receive consistent scheduling decisions from the leader.

Task Queue with Retries (using Kafka or RabbitMQ)

Store job requests in a distributed task queue.
Each node periodically polls the queue to fetch tasks.
Tasks are retried multiple times using exponential backoff if they fail, ensuring high availability and reliability.

Load Balancing and Throttling

Use a load balancer to distribute tasks evenly across nodes.
Implement rate limiting on task execution to avoid overwhelming the GPU cluster at certain times (e.g., midnight).

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using a consensus protocol for reliable scheduling, ensuring robust task queue management with retries, and implementing load balancing and throttling mechanisms, we can design a distributed scheduler that meets all the requirements. Key mistakes to avoid include not properly handling node failures or ignoring potential overloading of resources.

For follow-ups, one could discuss specific implementation details such as the choice of consensus protocol, the configuration of task queues, or the specifics of load balancing algorithms used.

**Comparison:**

```
Node	State	Current Term	Voter For	Log Index	Commit Index
1	Follower	0	None	0	0
2	Leader	1	Self	5	4
3	Follower	1	Self	0	0
```

**Code:**

```python
# Example Python code for a simple consensus-based scheduler using Raft

class Node:
    def __init__(self):
        self.state = "follower"
        self.current_term = 0
        self.voted_for = None
        self.log = []  # log of committed commands
        self.commit_index = 0
        self.last_applied = 0

def elect_leader(nodes):
    # Implement Raft leader election logic here
    pass

def schedule_job(node, job):
    if node.state == "leader":
        # Schedule the job for execution at a specific time
        pass

def run_task(task):
    # Execute the task and update logs
    pass
```

**Example input / output:**

```text
Input: Schedule Job for Team A at 2023-10-15T09:00Z

Output:
1. Leader receives request and schedules job.
2. Node 2 (leader) sends a scheduling message to Node 1.
3. Node 1 executes the task at 2023-10-15T09:00Z.

Input: Schedule Job for Team B at 2023-10-16T23:59Z

Output:
1. Leader receives request and schedules job.
2. Node 2 (leader) sends a scheduling message to Node 3.
3. Node 3 executes the task at 2023-10-16T23:59Z, throttling if necessary.
```

**Data:**

```json
{
  "nodes": [
    {"id": 1, "state": "follower"},
    {"id": 2, "state": "leader"},
    {"id": 3, "state": "follower"}
  ],
  "jobs": [
    {"id": 1, "team": "A", "time": "2023-10-15T09:00Z"},
    {"id": 2, "team": "B", "time": "2023-10-16T23:59Z"}
  ]
}
```

**Flow:**

1. Yes
2. Task Queue
3. Leader Node
4. Node Failure
5. Fallback to Next Leader
6. Recover from Failures
7. Throttling Mechanism
8. Load Balancer
9. Execute Task on AvailableNode

