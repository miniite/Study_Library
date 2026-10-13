---
type: "practice"
difficulty: "Deep"
---

# A stakeholder says 'let's just make it CP so it's always correct.' What real availability costs does that impose, and how do you decide per-endpoint whether CP or AP is right for your product?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Static Stability](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Static_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a stakeholder suggests prioritizing causal consistency (CP) over availability in distributed systems, they are asking us to focus on ensuring correctness at all costs. However, this comes with significant trade-offs that need careful consideration. In deciding per-endpoint whether to prioritize CP or eventual consistency (AP), we must evaluate the specific requirements and constraints of our system.

#### Triage
Understanding the Question

The question is essentially probing your ability to balance correctness and availability in a distributed system. The stakeholder's suggestion highlights a key principle in distributed systems architecture, namely that sacrificing some level of availability can ensure stronger consistency guarantees. However, this approach has real-world implications that must be carefully weighed.

#### Assumptions
and Context
Correctness vs. Availability: CP ensures all reads see the latest writes but may introduce latency or require retries.
Eventual Consistency (AP): AP allows for faster availability and lower latency but can lead to stale data.
Scalability Requirements: Different endpoints or services may have varying levels of tolerance for consistency issues.

#### First Principles Solution

To decide on CP vs. AP per-endpoint, we need to consider the following:

#### Cons
istency Requirements:

High transactional integrity: Systems handling financial transactions or sensitive data often require strong consistency.
Real-time analytics: Some services may need the latest data for real-time insights.

Latency Tolerances:

Low-latency systems like stock trading platforms cannot tolerate high latency.
High-latency systems can afford some level of staleness in their data.

Error Recovery and Redundancy:

Distributed systems with good redundancy and error recovery mechanisms may be able to handle AP more effectively.
Systems without robust redundancy might struggle under AP.
Step-by-Step Analysis

Identify Key Endpoints:

Determine which parts of the system require strong consistency (e.g., core financial transactions).
Identify sections where eventual consistency is acceptable (e.g., user interface updates).

Evaluate Trade-offs for Each Endpoint:

For critical endpoints, prioritize CP to ensure correctness.
For less-critical endpoints, lean towards AP for better availability and performance.

Implement Strategies:

Use techniques like optimistic concurrency control, distributed transactions (e.g., 2PC), or multi-version concurrency control (MVCC) for CP scenarios.
Employ caching mechanisms, read replicas, and other strategies to mitigate the impact of AP on user experience.

#### Example
Decision Making

#### Cons
ider a banking application with two main components:

Transaction Processing: This critical component handles deposits, withdrawals, and transfers. It must ensure every transaction is recorded correctly before it propagates to all nodes.

#### Cons
istency Requirement: High (CP).

#### Implementation
Strategy: Use distributed transactions like 2PC or optimistic concurrency control.

User Interface: This component provides a rich user experience but doesn't require immediate updates for every action.

#### Cons
istency Requirement: Lower (AP, with some staleness acceptable).

#### Implementation
Strategy: Implement caching and read replicas to improve response times.

#### Closing Summary

Balancing CP and AP in distributed systems is crucial. It requires a deep understanding of the system's requirements and constraints. By carefully analyzing each endpoint, we can make informed decisions that maximize both correctness and availability.

#### Common Mistakes/Follow-ups
Overly Prioritizing CP: Failing to consider performance implications can lead to suboptimal user experience.
Ignoring Scalability Concerns: Not accounting for the scalability of your chosen consistency model can limit future growth.
Inconsistent Decisions: Varying consistency levels across different components without a clear rationale can complicate system management.

Would you like to discuss any specific techniques or trade-offs in more detail?

**Comparison:**

```
Endpoint	Consistency Requirement	Implementation Strategy
Transaction Processing	High (CP)	Distributed Transactions (2PC)
User Interface	Lower (AP, with some staleness acceptable)	Caching and Read Replicas
```

**Code:**

```python
# Example of implementing optimistic concurrency control for CP scenarios

class Transaction:
    def __init__(self, account_id):
        self.account_id = account_id
        self.locked_version = 0

    def deposit(self, amount):
        # Acquire lock and check version before performing transaction
        with self.lock.acquire():
            if not self.check_version():
                return False
            self.balance += amount
            self.version += 1
            return True

    def check_version(self):
        current_version = get_account_version(self.account_id)
        if current_version != self.version:
            raise VersionMismatchError("Transaction failed due to version mismatch")
```

**Example input / output:**

```

Request:

PYTHON
Copy
transaction = Transaction("12345")
transaction.deposit(50)
print(transaction.balance)  # Expected output: 150 if version check passes, else raises VersionMismatchError

Response:

PLAINTEXT
Copy
150
```

**Data:**

```json
{
  "account_id": "12345",
  "balance": 100,
  "version": 1
}
```

**Flow:**

1. True
2. False
3. Transaction
4. Check Version
5. Commit Transaction
6. Retry Transaction

