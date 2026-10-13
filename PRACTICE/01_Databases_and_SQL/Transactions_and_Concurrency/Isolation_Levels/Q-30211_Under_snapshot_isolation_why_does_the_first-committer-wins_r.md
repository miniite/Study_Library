---
type: "practice"
difficulty: "Deep"
---

# Under snapshot isolation, why does the 'first-committer-wins' rule prevent lost updates but not write skew? Illustrate with two updates.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lost Updates and Write Skew](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Snapshot isolation is a database concurrency control mechanism that allows transactions to read data as of a snapshot taken at their start time, effectively isolating them from changes made by other transactions during execution. The "first-committer-wins" rule ensures that the first transaction to commit its writes wins in cases where multiple transactions attempt to modify conflicting rows. This rule prevents lost updates but not write skew because it does not handle concurrent modifications to different parts of the data model.

To illustrate, let's consider two transactions (T1) and (T2), each updating a row in a table with snapshot isolation enabled:

Initial State: The table has rows A(10) and B(20).

Transaction T1:

Reads A (value 10) at the start.
Modifies A to 30.
Tries to modify B to 50, but fails due to the rule because it is not the first to commit its changes.

Transaction T2:

Reads A and B (values are still 10 and 20 respectively) at the start.
Modifies B to 50.
Tries to modify A to 30, but fails due to the rule because it is not the first to commit its changes.

In this scenario, neither transaction can successfully write their updates, demonstrating how snapshot isolation and the "first-committer-wins" rule prevent lost updates by ensuring that only the first transaction to commit wins in cases of conflict. However, both transactions are blocked from making any writes, which is indicative of a write skew issue.

**Comparison:**

```
ID	Value
1	10
2	20
```

**Code:**

```sql
-- Example SQL for demonstration purposes (not intended to be executed)
BEGIN TRANSACTION;
UPDATE table SET A = 30 WHERE ID = 1; -- T1 starts and reads A as 10

BEGIN TRANSACTION;
UPDATE table SET B = 50 WHERE ID = 2; -- T2 starts and reads B as 20
```

**Example input / output:**

```text
- **Input**:
  - Initial state: `A(10)`, `B(20)`
  - Transaction T1 reads A (value 10)
  - Transaction T2 reads B (value 20)

- **Output**:
  - Neither transaction can update their respective values due to the "first-committer-wins" rule and snapshot isolation.
```

**Data:**

```json
{
  "table": [
    {"ID": 1, "Value": 10},
    {"ID": 2, "Value": 20}
  ]
}
```

**Flow:**

1. Transaction T1 Starts
2. Reads A=10
3. Attempts to Update A=30
4. T2 Starts
5. Reads B=20
6. Attempts to Update B=50
7. T2 Blocks on B Update, T1Blocks on A Update
8. Transaction Deadlock orInactive

