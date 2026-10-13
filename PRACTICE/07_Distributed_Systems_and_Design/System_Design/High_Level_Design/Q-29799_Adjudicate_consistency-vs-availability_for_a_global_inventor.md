---
type: "practice"
difficulty: "Expert"
---

# Adjudicate consistency-vs-availability for a global inventory system where overselling costs money but downtime costs more — where exactly do you place the CAP line per operation and why?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Payment Ledgers](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a high-level architecture (HLD) for a global inventory system, balancing consistency versus availability is crucial, especially when both overselling costs money but downtime costs even more. The CAP theorem suggests that in distributed systems, you can only guarantee two out of the three properties: Consistency, Availability, and Partition Tolerance. Our task here is to determine where exactly we place this line for various operations.

#### Triaging the Question

The question asks us to adjudicate consistency versus availability for a global inventory system where:

Overselling (selling more than available stock) costs money.
Downtime costs more than overselling.

This means that while it's costly to sell beyond what we have, it’s even costlier to have downtime where customers cannot access the system at all. Therefore, our design needs to prioritize availability over consistency in certain critical scenarios.

#### First-Principles Solution

Let's break down the decision process step-by-step:

Understand the Operations:

Read Operations: Get current stock levels.
Write Operations: Update stock levels upon a sale.
Delete Operations: Remove items from inventory when they are sold.

Determine the Impact of Each Operation on Consistency and Availability:

Read Operations (Current Stock Levels):

#### Cons
istency Requirement: High, as stale data can lead to overselling.
Availability Requirement: Moderate to high, but not critical if there's a brief delay.

Write Operations (Updating Stock After Sale):

#### Cons
istency Requirement: High, to avoid double-counting or incorrect stock levels.
Availability Requirement: Critical, as customers rely on this data for real-time purchase decisions. A slight delay is acceptable but not a complete failure.

Delete Operations (Removing Sold Items):

#### Cons
istency Requirement: High, ensuring the system reflects the correct sold status.
Availability Requirement: Moderate to high, depending on how critical it is to quickly reflect changes in the global inventory.

Strategic Placement of CAP Line:

For Read Operations and Write Operations, we should prioritize availability over consistency where possible. This means accepting some degree of inconsistency (e.g., stale read) if it ensures that all users can access the system without downtime.

Read Operation
Accept Stale Read
Write Operation
Ensure Write-Ahead Logging

For Delete Operations, while availability is still important, we might be willing to accept a slight delay in consistency if it prevents complete system failure. This means using techniques like eventual consistency or optimistic locking.

Mitigating Risk:

Use techniques such as:
Eventual Consistency: Ensure that all nodes eventually converge on the correct state.
Optimistic Concurrency Control (OCC): Allow writes but detect and resolve conflicts later if they arise.
Write-Ahead Logging (WAL): Ensure transactions are recorded before committing to prevent data loss.

#### Closing Summary

In summary, for our global inventory system:

We prioritize availability over consistency during read and write operations by allowing stale reads and ensuring eventual consistency.
For delete operations, while we still prefer consistency, we can tolerate a slight delay if it prevents downtime.

This approach balances the cost of overselling with the higher cost of downtime to ensure that users have access to the system even if there are short-term inconsistencies in inventory levels.

#### Common Mistakes
and Follow-Ups
Mistake 1: Prioritizing consistency over availability without considering the costs associated with each.

#### Follow-Up
1: Discussing how to implement a hybrid approach using techniques like eventual consistency or optimistic concurrency control.
Mistake 2: Ignoring the impact of partition tolerance on global systems, which is always necessary in distributed environments.

#### Follow-Up
2: Explaining how to design for network partitions and ensuring that the system remains operational during such events.

**Code:**

```
MERMAID
Copy
graph TD;
  A[Read Operation] --> B[Accept Stale Read];
  C[Write Operation] --> D[Ensure Write-Ahead Logging];
```

