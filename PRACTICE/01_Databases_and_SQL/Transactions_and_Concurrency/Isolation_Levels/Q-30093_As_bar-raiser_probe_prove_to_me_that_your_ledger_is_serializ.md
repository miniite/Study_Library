---
type: "practice"
difficulty: "Expert"
---

# As bar-raiser probe: prove to me that your ledger is serializable end-to-end when it uses SSI in Postgres but the app-side outbox pattern commits in a separate transaction.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Serializable Performance](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Serializable_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our ledger remains serializable end-to-end, especially given the use of SSI (Single Statement Isolation) in PostgreSQL and the app-side outbox pattern that commits in a separate transaction, we need to carefully analyze how these components interact and consider the implications on isolation levels. This discussion will explore the concept of serializability, examine the role of SSI and the outbox pattern, and finally outline the steps required to maintain consistency.

**Comparison:**

| Transaction ID | Type     | Data                     |
|----------------|----------|--------------------------|
| 1              | outbox   | {"amount": 50, "action": "deposit"} |
| 2              | ledger_entry | {"account_id": 42, "balance": 100}  |

**Code:**

```sql
-- Example SQL for setting transaction isolation level in PostgreSQL
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

**Example input / output:**

```text
Input:
- Initial state of the ledger with account 42 having a balance of $100.
- Outbox transaction ID 1 for depositing $50 into account 42.

Output (desired behavior):
- After processing outbox transaction, ensure that the final state of the ledger matches serializable outcomes, i.e., no two transactions violate any constraints if they were executed in a single serial transaction.
```

**Data:**

```json
{
  "transactions": [
    {
      "id": 1,
      "type": "outbox",
      "data": {"amount": 50, "action": "deposit"}
    },
    {
      "id": 2,
      "type": "ledger_entry",
      "data": {"account_id": 42, "balance": 100}
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initial Ledger State
4. Is Outbox Transaction?
5. Process Outbox
6. Check Serializability
7. Direct Ledger Entry
8. Check Serializability

