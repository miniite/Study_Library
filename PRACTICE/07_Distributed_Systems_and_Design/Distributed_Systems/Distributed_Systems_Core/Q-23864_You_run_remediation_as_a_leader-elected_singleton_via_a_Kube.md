---
type: "practice"
difficulty: "Deep"
---

# You run remediation as a leader-elected singleton via a Kubernetes lease, but during a stop-the-world GC pause a second worker acquired the lease and both ran the same runbook. Explain the safety gap in lease-based election and how a fencing token prevents the double-run.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Leader_Election.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, particularly in AIOps where real-time monitoring and automated remediation are critical, ensuring that only one instance runs a specific task at any given time is crucial to prevent race conditions and ensure data integrity. The lease-based leader election mechanism using Kubernetes (K8s) can be effective for this purpose, but it has inherent safety gaps during certain operations like stop-the-world garbage collection (GC).

The core issue here is that while the leader election ensures a single instance holds the lease at any given moment under normal operation, it does not provide fail-proof protection against double-runs during critical system states. Specifically, if a GC pause occurs and another worker manages to acquire the lease before the first one can release it, both instances might execute the same remediation runbook, leading to unintended consequences.

To address this safety gap, Kubernetes introduces the concept of fencing tokens. These tokens act as a form of atomic flag that ensures only one instance is active at any given time by providing an additional layer of validation beyond just the lease.

Here’s how it works in detail:

Leader Election and Lease: A worker node attempts to acquire a lease, which essentially means becoming the leader for executing specific tasks.
GC Pause and Double-Run Scenario: During a stop-the-world GC pause, if another worker acquires the lease before the current leader can release it, both instances might proceed to execute the same remediation task simultaneously.
Fencing Tokens Solution: Each time a node becomes the leader, it not only acquires a lease but also generates and broadcasts a unique fencing token. The next time any instance attempts to become the leader, it checks if the current token matches the one it received from the previous leader. If they don’t match, it knows that another instance is already holding the lease and should not proceed.

Let’s break down this solution step by step:

Initial Leader Election: A node successfully acquires a lease and generates a unique fencing token.
Token Distribution: The token is distributed to other nodes or stored in an accessible location (like Kubernetes stateful sets).
Periodic Check for Token: When another instance wants to become the leader, it checks if the current token matches the one it received. If they don’t match, it skips the leadership process.
Token Refresh and Update: The leader periodically refreshes the token and updates its distribution to ensure continuous validation.

By incorporating fencing tokens into the lease-based election mechanism, Kubernetes can effectively prevent double-runs during critical system states like GC pauses. This additional layer of validation ensures that only one instance is active at any given time, thereby mitigating the safety gap present in traditional leader election mechanisms.

**Comparison:**

```
Node	Lease Expires At	Current Token
A	2023-10-10T10:00:00Z	token_456
B	2023-10-10T10:30:00Z	token_789
```

**Code:**

```python
# Example Pseudocode for Token Validation During Lease Election

def validate_leader_election():
    current_token = get_current_fencing_token()
    if not token_matches(current_token):
        raise LeaderElectionFailed("Token mismatch, another instance is the leader")
```

**Example input / output:**

```text
Input: Node A attempts to become the leader while Node B already holds a lease with token "token_456".
Output: Node A checks its fencing token and realizes it doesn't match, thus refrains from taking action.
```

**Data:**

```yaml
# Sample Fencing Tokens and Leadership Data
fencingTokens:
  - token: "token_123"
  - token: "token_456"

leaderElection:
  nodeA:
    leaseExpiresAt: "2023-10-10T10:00:00Z"
    currentToken: "token_123"
  nodeB:
    leaseExpiresAt: "2023-10-10T10:30:00Z"
    currentToken: ""
```

**Flow:**

1. Check Token
2. No
3. Yes
4. Node A
5. Token Match?
6. Refrain from Action
7. Claim Leadership

