---
type: "practice"
difficulty: "Expert"
---

# You're deciding whether a new globally distributed feature should use a CP database that rejects writes on the minority side of a partition or an AP store that accepts them and reconciles later. Give the per-endpoint framework you'd hand product teams so they choose correctly without being distributed-systems experts.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed system, ensuring data consistency versus availability is a fundamental trade-off. When deciding between CP (Consistency, Partition Tolerance) and AP (Availability, Partition Tolerance) databases, it's crucial to understand how they handle network partitions—specifically, whether writes are rejected or accepted during a split.

Summary: To choose the right database for a new globally distributed feature, we need to design a framework that balances consistency and availability based on the specific use case. We will outline criteria like data integrity requirements, failure modes, and recovery strategies to guide non-expert product teams through this decision-making process.

#### Triage
The question is asking us to create a decision framework for selecting between CP (writes rejected) and AP (writes accepted) databases. The goal is to provide clear guidance without overwhelming the user with technical complexities. Key assumptions include:

Network partitions will occur.
The system must handle these partitions gracefully while balancing consistency and availability.

#### First Principles Solution

Identify the Consistency Requirements: Determine if strong consistency (CP) or eventual consistency (AP) aligns better with your use case.
Understand Failure Modes: Assess how data integrity is affected by network failures.
Define Recovery Strategies: Plan for how to recover from partitions and ensure service continuity.

Let's walk through each step in detail:

Identify the Consistency Requirements:

Transaction Integrity: Does your application require all transactions to be committed or rolled back atomically, even during network partitions?
Data Accuracy: Must data be accurate and up-to-date across all nodes at all times?

Understand Failure Modes:

Network Partitions: How often are you likely to experience network failures? What is the acceptable downtime for your application in these scenarios?
Write Acceptance During Partition: Will rejecting writes during a partition cause significant issues, or can you accept temporary inconsistencies and handle them later?

Define Recovery Strategies:

Data Reconciliation Mechanisms: If using an AP database, how will data be reconciled after the network partitions resolve?
Rollback and Retry Policies: How will transactions be handled if they fail due to a partition? Should there be automatic retries or manual intervention?

#### Example
Criteria Framework:

Criteria	CP Database	AP Database
Data Integrity During Partition	Rejects writes, ensuring atomicity	Accepts writes, risk of temporary inconsistency
Failure Tolerance	High (atomic transactions)	Lower (tolerates some data loss)
Recovery Time Objective (RTO)	Low (rapid recovery required for consistency)	Higher (reconciliation can take time)
Operational Complexity	Higher (requires careful partition handling and rollback strategies)	Lower (simpler operational model)
Cost	Potentially higher due to additional redundancy and handling mechanisms	Lower, but risk of data loss during partitions

#### Closing Summary
This framework provides a structured approach for non-expert product teams to evaluate the trade-offs between CP and AP databases. By focusing on consistency requirements, failure modes, and recovery strategies, they can make an informed decision tailored to their specific use case.

#### Common Mistakes/Follow-Ups

Ignoring Recovery Strategies: For AP databases, ensure there are robust reconciliation mechanisms in place.
Overlooking Consistency Requirements: CP databases might be over-engineered if the application does not strictly require strong consistency.
Neglecting Performance Considerations: Both models have implications on performance and scalability; consider these factors during decision-making.

**Comparison:**

```
Criteria	CP Database	AP Database
Data Integrity During Partition	Rejects writes, ensuring atomicity	Accepts writes, risk of temporary inconsistency
Failure Tolerance	High (atomic transactions)	Lower (tolerates some data loss)
Recovery Time Objective (RTO)	Low (rapid recovery required for consistency)	Higher (reconciliation can take time)
Operational Complexity	Higher (requires careful partition handling and rollback strategies)	Lower (simpler operational model)
Cost	Potentially higher due to additional redundancy and handling mechanisms	Lower, but risk of data loss during partitions
```

**Code:**

```

-- Example of a CP database query to handle partition failures BEGIN TRANSACTION; INSERT INTO Orders (CustomerID, ProductID) VALUES (12345, 67890); IF (NetworkPartitionDetected()) THEN ROLLBACK; ELSE COMMIT; END IF;

CODE
Copy

```sql
-- Example of an AP database query accepting writes during partition
INSERT INTO Orders (CustomerID, ProductID) VALUES (12345, 67890);
```

**Example input / output:**

```

Input:

Network partition detected between nodes.
Write request from Node 1 to insert an order.

Output (CP Database):

SQL
Copy
INSERT INTO Orders (CustomerID, ProductID) VALUES (12345, 67890);
-- Transaction is rolled back due to network partition

Rollback Operation:

Output (AP Database):

SQL
Copy
INSERT INTO Orders (CustomerID, ProductID) VALUES (12345, 67890);
-- Write is accepted, but reconciliation will occur later
Reconciliation Operation:
```

**Data:**

```json
{
  "partition": {
    "node1": ["OK", "OK"],
    "node2": ["Failed"]
  }
}
```

**Flow:**

1. Reject Write
2. Accept Write
3. Network Partition Detected
4. CP Database
5. Rollback
6. AP Database
7. Reconciliation

