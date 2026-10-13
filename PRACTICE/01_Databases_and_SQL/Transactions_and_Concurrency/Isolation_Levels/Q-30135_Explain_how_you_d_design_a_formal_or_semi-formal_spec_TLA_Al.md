---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design a formal or semi-formal spec (TLA+/Alloy) of a critical transaction to prove the chosen isolation level actually prevents the anomaly you care about.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation for Shared Subsystems](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_for_Shared_Subsystems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a formal specification using TLA+ (Temporal Logic of Actions) or Alloy to ensure that a specific transaction adheres to a chosen isolation level and prevents the desired anomalies, we first need to understand the context and the isolation level in question. For this example, let's consider ensuring that a transaction under RC (Read Committed) isolation level prevents the dirty read anomaly. Here’s how you would approach this:

Context: We are dealing with a database system where transactions manipulate shared data, and we need to ensure they do not cause dirty reads, which occur when one transaction sees uncommitted changes from another.
Isolation Level: RC isolation level ensures that transactions see only committed changes but might read "dirty" (uncommitted) values.

Now, let’s delve into the formal specification using TLA+ and Alloy:

#### Triage

The core of the question is to design a formal or semi-formal specification that rigorously proves that a transaction under RC isolation will not result in dirty reads. We need to define the system state, actions, and transitions that ensure the integrity of data access.

#### First-Principles Solution
1. Define System Components

First, we define the components of our database: transactions, shared data items, and their states (committed or not).

state diagram
    node Committed
    node Uncommitted

    transition to Uncommitted from Committed
2. Define Actions and Transitions

We need to specify actions such as read and write, along with transitions that ensure data integrity under RC.

TLA
VARIABLES data, transactionStatus, transactionActions

Init == <data={Uncommitted}, transactionStatus={}>

Next ==
    \E t: transactions :
        (transactionStatus[t] = Uncommitted & \A d: data :
            (transactionActions[t,d] ->
                (transactionStatus[t]' = Committed) &
                (data[d]' = committedValue)
            )
        ) |
        (transactionStatus[t] = Committed & transactionActions[t,d] -> data[d]' = data[d])
3. Ensure Read Actions Prevent Dirty Reads

A read operation should only see committed values, ensuring no uncommitted changes are read.

TLA
Read(t,d) == transactionActions[t,d]
Write(t,d,v) == (transactionStatus[t] = Uncommitted & \A d: data :
    (Read(t,d) -> v = committedValue)
)

// Ensure that all reads see only committed values
\A t: transactions, d: data :
    Read(t,d) -> transactionStatus[d] = Committed
4. Define Transitions for Committing Transactions

Transactions can commit their changes if they have read from and written to the same state.

TLA
Commit(t) == \E d: data : (Read(t,d) & Write(t,d,committedValue))

#### Closing Summary

By defining these components and transitions using TLA+, we ensure that a transaction under RC isolation will only see committed values. The key idea is to enforce that all read operations must occur after the last write operation by the same transaction, preventing dirty reads.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to properly define and transition between committed and uncommitted states.

#### Follow-Up
Ensuring that all read actions respect the state of transactions.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Transaction Workflow under RC Isolation
    dateFormat YYYY-MM-DD

    section Transitions
        transaction1: T1 -> Commit(d), from 2023-10-01 to 2023-10-03, effort 2d
        transaction2: T2 -> Read(d), from 2023-10-02 to 2023-10-04, effort 3d

    section Events
        T1_write_d: Write(d, committedValue)
```

**Code:**

```
TLA+
Copy
VARIABLES data, transactionStatus, transactionActions

Init == <data={Uncommitted}, transactionStatus={}>

Next ==
    \E t: transactions :
        (transactionStatus[t] = Uncommitted & \A d: data :
            (transactionActions[t,d] ->
                (transactionStatus[t]' = Committed) &
                (data[d]' = committedValue)
            )
        ) |
        (transactionStatus[t] = Committed & transactionActions[t,d] -> data[d]' = data[d])

Read(t,d) == transactionActions[t,d]

Write(t,d,v) == (transactionStatus[t] = Uncommitted & \A d: data :
    (Read(t,d) -> v = committedValue)

Commit(t) == \E d: data : (Read(t,d) & Write(t,d,committedValue))

\A t: transactions, d: data :
    Read(t,d) -> transactionStatus[d] = Committed
```

**Example input / output:**

```text
sequenceDiagram
    participant Transaction1 as T1
    participant Database as DB

    T1 ->> DB: read(d)
    activate T1
    DB -->> T1: uncommittedValue (T1 is still writing)
    deactivate T1
    T1 ->> DB: write(d, committedValue)
    T1 ->> DB: commit(d)
```

**Data:**

```json
{
  "data": {
    "Uncommitted": {},
    "Committed": {}
  },
  "transactionStatus": {},
  "transactionActions": {}
}
```

**Flow:**

1. Transaction Starts
2. Read d
3. Commit d
4. Write d, CommittedValue
5. Transaction Ends

