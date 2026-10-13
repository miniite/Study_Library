---
type: "practice"
difficulty: "Expert"
---

# Two teams independently built distributed-lock libraries with subtly different fencing semantics, and a cross-service flow that composes them corrupts state. As the architect, how do you resolve the dispute, converge the standard, and migrate both without a big-bang cutover?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Coordination Choices](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As an experienced backend architect, when faced with this scenario, my approach would involve several steps to resolve the dispute between the two teams, converge on a standard fencing semantics, and migrate both libraries without causing downtime or corruption of state. The key is to deeply understand the root causes of the race conditions, align the teams on best practices, and implement a migration plan that minimizes disruption.

Firstly, it's crucial to clarify what "fencing semantics" means in this context. Fencing refers to mechanisms used by distributed systems to ensure mutual exclusion or synchronization among processes. Common fencing methods include lease-based locks, token-passing protocols, and distributed consensus algorithms like Raft or Paxos. The subtle differences between the two libraries likely stem from variations in how they handle these mechanisms.

To resolve this issue, we need a structured approach:

Understand the Current State: Gather detailed information about both libraries, including their design documents, codebases, and implementation details.
Identify the Dispute: Determine which aspects of fencing semantics differ between the two libraries.
Analyze Impact: Assess the potential impact on existing services that rely on these libraries to ensure state consistency.
Converge on a Standard: Develop a unified standard for fencing semantics, leveraging best practices from both implementations.
Migrate Services Gradually: Plan and execute a migration strategy that minimizes disruption.

Let's break this down step by step:

Step 1: Understand the Current State
Review both libraries' codebases to identify their fencing mechanisms and assumptions.
Analyze logs and trace data from services that use these libraries to understand any existing issues or inconsistencies.
Identify the key differences in how each library handles lock acquisition, release, and propagation.
Step 2: Identify the Dispute
Document the specific differences in fencing semantics. For example:
Library A uses a time-based lease mechanism, while Library B uses an event-driven token-passing approach.
Library A relies on distributed consensus for each lock request, whereas Library B uses simpler heartbeat mechanisms.
Step 3: Analyze Impact
Assess the impact of these differences on state consistency. Consider edge cases such as:
Deadlocks or livelocks due to misaligned locking patterns.
Inconsistent states if one library releases a lock prematurely while another service is still using it.
Performance implications, especially in high-concurrency scenarios.
Step 4: Converge on a Standard
Based on the analysis, choose a standardized approach that minimizes inconsistencies and aligns with best practices. For example:
Select a common locking mechanism (e.g., lease-based) and ensure both libraries use it consistently.
Implement distributed consensus algorithms only where necessary to avoid unnecessary complexity.
Step 5: Migrate Services Gradually
Develop a migration plan that allows services to transition from one library to another without causing disruptions:
Parallel Operation: Run both versions of the service for a period to gather data on any issues.
A/B Testing: Use A/B testing to ensure new state consistency rules are met before full deployment.
Rollout Strategy: Gradually roll out changes, starting with non-critical services and moving to critical ones.

#### Closing Summary

In conclusion, resolving this dispute involves a thorough understanding of both libraries' design choices, aligning on best practices, and executing a meticulous migration strategy. By taking a methodical approach, we can ensure minimal disruption and maintain state consistency across the system.

#### Common mistakes
to avoid include:

Overlooking edge cases that could lead to race conditions.
Failing to test thoroughly during the migration phase.
Rushing through changes without adequate planning and rollback mechanisms.

**Comparison:**

```
Lock Mechanism	Consensus Algorithm?	Use Case
Lease-based	No	Resource
Token-passing	Yes	State
```

**Code:**

```python
# Example of a simple distributed lock implementation using Python's threading module for demonstration purposes
import threading
from contextlib import contextmanager

class DistributedLock:
    def __init__(self, name):
        self.name = name
        self.lock = threading.Lock()

    @contextmanager
    def acquire(self):
        # Simulate acquiring a distributed lock
        print(f"Acquiring {self.name}")
        with self.lock:
            yield f"Locked: {self.name}"
        print(f"Releasing {self.name}")
```

**Example input / output:**

```text
Input:
- ServiceA uses lease-based lock with no consensus algorithm.
- ServiceB uses token-passing lock with distributed consensus.

Output:
- Potential race conditions in state synchronization if both services use different fencing semantics.
- Identified issues include deadlocks, inconsistent states during release, and performance bottlenecks.

Dry Run:
1. Simulate concurrent requests to services using both libraries.
2. Identify race conditions by observing log data.
3. Propose a unified approach based on identified best practices.
```

**Data:**

```json
{
  "serviceA": {
    "lockMechanism": "lease-based",
    "consensusAlgorithm": false,
    "useCase": "resource allocation"
  },
  "serviceB": {
    "lockMechanism": "token-passing",
    "consensusAlgorithm": true,
    "useCase": "state synchronization"
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Differences
4. Converge Standard?
5. Develop Unified Approach
6. Migrate Services Gradually
7. Implement and Test
8. Test Parallel Operation
9. Roll Out Changes

