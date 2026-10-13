---
type: "practice"
difficulty: "Deep"
---

# Describe write skew with a concrete example (e.g. on-call scheduling). Which isolation levels allow it and which prevent it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lost Updates and Write Skew](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Write skew is a specific type of concurrency issue that can occur when multiple transactions are accessing and modifying related data in such a way that they interfere with each other. In the context of on-call scheduling, this might happen if you have two or more teams attempting to update their schedules simultaneously. For example, if Team A's schedule overlaps with another team’s critical period, it could cause issues.

To illustrate write skew, let's consider an on-call scheduling system where each team has a specific window for taking shifts. We will use the isolation levels RU (Read Uncommitted), RC (Read Committed), RR (Repeatable Read), and Serializable to see which ones prevent or allow this issue.

**Code:**

```
MERMAID
Copy
graph TD;
    subgraph TeamA[Team A Scheduling]
        A1[Start Shift] --> A2[End Shift]
        A3[Save Schedules]
    end

    subgraph TeamB[Team B Scheduling]
        B1[Start Shift] --> B2[End Shift]
        B3[Save Schedules]
    end
```

**Example input / output:**

```text
- **Initial State:** Both Team A and Team B have their on-call shifts defined.
- **Team A starts a transaction to update its shift times.** It sets the new start time to `09:00`.
- **Before Team A's transaction commits, Team B starts a transaction to update its own shift times.** Team B changes the end time for its team to `15:00`.

Given these operations:
- In **RU (Read Uncommitted)** isolation level, both transactions can read uncommitted data. Therefore, when Team B reads from the database, it sees the new start time of 09:00 as committed by Team A before it has been officially saved.
- After committing, Team B sets its end time to `15:00`, overlapping with Team A's new shift.

- In **RC (Read Committed)** isolation level, a transaction can only read data that is committed. If Team B starts after Team A commits its changes, it will see the updated start time of 09:00 and not overlap. However, if both transactions start almost simultaneously, there’s a race condition.

- In **RR (Repeatable Read)** isolation level, once a transaction locks data for reading, it retains those locks until the end of the transaction. If Team A starts its transaction first and locks the entire shift times, Team B would not be able to change overlapping shifts during this period.

- **Serializable** is the strictest isolation level where all transactions are serialized, ensuring that no two transactions can overlap. This guarantees that only one transaction at a time can modify the data.
```

**Data:**

```json
{
  "teams": {
    "A": { "shift_start": "08:00", "shift_end": "16:00" },
    "B": { "shift_start": "14:00", "shift_end": "22:00" }
  }
}
```

