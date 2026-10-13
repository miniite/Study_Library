---
type: "practice"
difficulty: "Deep"
---

# Your Q&A reputation computation is recalculated on every page load and is now the slowest query; how do you redesign so reputation is maintained incrementally and stays consistent after a vote is retracted?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Maintaining Denormalized Views](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the slow recalculation of Q&A reputation, we need to redesign our system to maintain reputation incrementally. This involves ensuring that reputation changes are tracked in real-time and can be efficiently rolled back if needed. Here’s a step-by-step solution from first principles.

The core idea is to implement a distributed, low-level design (LLD) where:

Reputation updates are logged as transactions.
A global consensus mechanism ensures consistency across the system.
Incremental updates minimize database contention and reduce overall query latency.

First, let’s triage the problem: The current design recalculates reputation on every page load, which is inefficient and slow. We need to optimize this by ensuring that reputation changes are handled atomically and can be retraced if necessary. This requires a detailed understanding of our system architecture, data flow, and consensus protocols.

#### Assumptions

Our system has a distributed nature with multiple servers.
Users can vote up or down on questions and answers.
We need to maintain real-time reputation updates across the entire system.
Retracting votes must be possible without full recalculation.

Now, let’s dive into the solution:

Step 1: Transaction Logging

We will introduce a ReputationTransaction table that logs every vote. Each entry in this table includes:

Vote ID
Voter ID
Target (Question/Answer) ID
Type of vote (up/down)
Timestamp

This log allows us to reconstruct the reputation state at any point in time and roll back changes if needed.

Step 2: Real-Time Reputation Calculation

Instead of recalculating reputation on every page load, we implement a real-time calculation using a distributed consensus protocol like Raft or Paxos. This protocol ensures that all nodes in our system agree on the latest reputation values.

Design Pattern: Eventual Consistency with Strong Leadership

We use an approach where:

A single leader node handles vote transactions.
Other nodes replicate and validate these transactions.
Once a transaction is validated, it is applied to the local state.
Step 3: Incremental Reputation Update

When a vote occurs, the leader node immediately updates the ReputationTransaction table. Simultaneously, it starts an incremental update process that:

Reads the current reputation from the database.
Applies the new transaction incrementally.
Updates the local state.

This ensures minimal database contention and fast response times for users.

Step 4: Retraction of Votes

If a vote is retracted, we simply log another ReputationTransaction that negates the original change. The system then re-runs the incremental update process to reflect this retraction accurately.

#### Example
Workflow:
User votes up on question Q.
Leader node logs transaction (Q, user_id, up).
Leader node starts an incremental update: reads current reputation, applies +1 for vote up, updates local state.
Other nodes replicate and validate the transaction.
For retraction: User retracts their vote.
Leader logs another transaction (Q, user_id, down) to negate the previous change.
Incremental update runs again with -1 applied.
Step 5: Monitoring and Debugging

Implement monitoring tools to track reputation changes in real-time. Use metrics like latency, success rate of transactions, and consistency across nodes to ensure system health.

#### Closing Summary

By redesigning our reputation computation using a distributed transaction log and incremental updates, we can significantly reduce the overhead of recalculating reputations on every page load. This approach ensures that reputation is always up-to-date and maintainable even after votes are retracted.

#### Common Mistakes/Follow-Ups

Not accounting for race conditions in real-time updates.
Failing to implement a strong leader node for consensus.
Ignoring the need for replication and validation of transactions across nodes.
Overlooking the importance of monitoring and debugging tools.

**Comparison:**

```
table "ReputationTransaction" as rt {
    id Integer
    voter_id Integer
    target_id Integer
    vote_type String @default("up")
    timestamp DateTime
}

table "ReputationState" as rs {
    target_id Integer
    reputation Integer
}
```

**Code:**

```python
# Example Python code snippet for handling votes
class ReputationService:
    def __init__(self):
        self.transactions = []
        self.reputation_state = {}

    def handle_vote(self, voter_id, target_id, vote_type):
        transaction = {
            'voter_id': voter_id,
            'target_id': target_id,
            'vote_type': vote_type
        }
        # Log the transaction
        self.transactions.append(transaction)

        # Incrementally update reputation state
        if vote_type == 'up':
            self.reputation_state[target_id] += 1
        elif vote_type == 'down':
            self.reputation_state[target_id] -= 1

    def retract_vote(self, voter_id, target_id):
        # Log the retraction transaction
        retract_transaction = {
            'voter_id': voter_id,
            'target_id': target_id,
            'vote_type': 'undo'
        }
        self.transactions.append(retract_transaction)

        if 'up' in self.reputation_state[target_id]:
            self.reputation_state[target_id] -= 1
        elif 'down' in self.reputation_state[target_id]:
            self.reputation_state[target_id] += 1

    def get_reputation(self, target_id):
        return self.reputation_state.get(target_id, 0)
```

**Example input / output:**

| Input                | Output            |
|----------------------|-------------------|
| handle_vote(1, 123, 'up') | Reputation state for QID 123 is now +1 |
| retract_vote(2, 123)   | Reputation state for QID 123 is now -1 (reverted to initial state) |

**Data:**

```json
{
  "transactions": [
    {
      "voter_id": 1,
      "target_id": 123,
      "vote_type": "up"
    },
    {
      "voter_id": 2,
      "target_id": 123,
      "vote_type": "down"
    }
  ],
  "reputation_state": {
    "123": 0
  }
}
```

**Flow:**

1. User Votes
2. Leader Node
3. Log Transaction
4. Start Incremental Update
5. Replicate and Validate
6. Update Local State
7. Retract Vote
8. Leader Node
9. Log Retraction Transaction
10. Re-run Incremental Update

