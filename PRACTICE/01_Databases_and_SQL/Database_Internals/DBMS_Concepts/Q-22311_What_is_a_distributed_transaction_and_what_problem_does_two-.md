---
type: "practice"
difficulty: "Deep"
---

# What is a distributed transaction and what problem does two-phase commit try to solve?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Distributed Transactions and 2PC](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the world of distributed systems, transactions are critical for ensuring data consistency and integrity across multiple databases or nodes. A distributed transaction refers to a series of actions that must be executed as a single unit of work, with all parts succeeding or failing together. If any part fails, the entire transaction should be rolled back to maintain consistency.

The two-phase commit (2PC) protocol is designed to ensure the correctness and atomicity of distributed transactions in a highly reliable manner. It addresses the challenge of ensuring that multiple databases or services agree on whether to commit or abort a transaction when changes are made across them.

#### Triaging the Question
What does 2PC solve? The main problem 2PC solves is achieving consistency and atomicity in distributed transactions, where multiple parties need to coordinate their actions.

#### Assumptions
We assume we have a system with multiple nodes or databases that need to participate in a transaction. These nodes might be geographically dispersed and communicate via messages (e.g., RPCs).
Key Idea: 2PC ensures that all participants either commit the transaction or roll it back, maintaining consistency even if some nodes fail.

#### First-Principles Solution
Phase 1: Preparation
Coordinator Node Initialization: A coordinator node initiates a transaction and sends prepare messages to each participant node.
Participant Response: Each participant checks its local state and responds with either "prepared" (if it can commit) or "not prepared" (if it cannot).
Phase 2: Commit or Abort

Coordinator Decision: After receiving all responses, the coordinator makes a decision:

If at least one node responded "not prepared," the transaction is aborted.
Otherwise, the transaction is committed.

Participant Action: Once the coordinator decides, it sends commit messages to participants if they are ready to commit or abort messages if not.

Handling Failures
Coordinator Failure: In case of a failure at any point before sending the final decision, a recovery process can handle resending messages.
Participant Failure: If a participant fails during preparation, it may need to be marked as "not prepared" in subsequent rounds.

#### Closing Summary
and Common Mistakes/Follow-Ups

2PC is a robust solution for distributed transactions but comes with trade-offs:

Latency: The process introduces additional latency due to the two round-trips required.
Failure Modes: If a participant cannot communicate (e.g., network partition), 2PC might fail to reach a decision, leading to inconsistencies.

#### Common mistakes
include:

Overlooking edge cases like network partitions or node failures.
Not implementing proper recovery mechanisms.
Triaging Follow-Ups
How does 2PC compare with other protocols? Discuss alternatives like one-phase commit, three-phase commit, and how each handles different failure scenarios.
Scalability in a large distributed system. Explore the scalability challenges of 2PC and discuss optimizations or alternative approaches.

**Comparison:**

```
Phase	Actions	Decision
Preparation	Send prepare messages, check readiness	No action taken yet
Commit	Send commit messages if all participants are prepared	All transactions committed
Abort	Send abort messages to unprepared participants	Transaction aborted
```

**Code:**

```python
# Example simplified two-phase commit protocol flow

class Node:
    def __init__(self):
        self.state = 'not prepared'

    def prepare(self, transaction_id):
        # Check local state for the transaction
        if can_commit(transaction_id):
            self.state = 'prepared'
            return True
        else:
            return False

def two_phase_commit(transactions):
    coordinator = Node()

    # Phase 1: Preparation
    for participant in participants:
        if not participant.prepare(transaction_id):
            abort_transaction(coordinator, transaction_id)
            break

    # Phase 2: Commit or Abort
    if all(participant.state == 'prepared' for participant in participants):
        commit_transaction(coordinator, transaction_id)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Node A prepares transaction TX123 (prepared) | Node A state changes to 'prepared' |
| Node B prepares transaction TX123 (not prepared due to local constraints) | Node B state remains 'not prepared' and aborts the transaction |
| Coordinator makes decision based on responses | Transaction is aborted |

**Data:**

```json
{
  "transaction": {
    "id": "TX123",
    "participants": ["nodeA", "nodeB", "nodeC"],
    "status": "not prepared"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Transaction Initiation
6. Prepare Phase
7. All Prepared?
8. Commit Transactions
9. Abort Transactions
10. Participant Failure?
11. Recover and Retry

