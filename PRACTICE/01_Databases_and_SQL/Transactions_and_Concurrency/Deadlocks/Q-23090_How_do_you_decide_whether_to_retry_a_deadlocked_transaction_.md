---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide whether to retry a deadlocked transaction automatically or surface the error to the user?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Retry Strategy](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Retry_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with deadlock detection and resolution, the decision between retrying a deadlocked transaction automatically versus surfacing an error to the user depends on the specific requirements of the application, its context, and the potential impact of both approaches. The core idea is to balance performance, reliability, and user experience.

#### Triaging the question
reveals that the interviewer is probing your understanding of deadlock scenarios, the implications of automatic retries vs. explicit errors, and how these choices affect system behavior. Key assumptions include the transaction's criticality, the frequency of deadlocks, and whether retrying would solve the issue or simply mask it.

First-principles reasoning starts with recognizing that a deadlock occurs when two or more transactions are waiting for resources held by each other, leading to a circular wait condition. Deadlock detection involves identifying these cycles and deciding on an appropriate resolution strategy.

Automatic Retry Mechanism:

#### Pros
It can help the system recover from deadlocks without human intervention, ensuring high availability and reliability.

#### Cons
Frequent retries can lead to unnecessary resource consumption and potential performance degradation. Retrying might also mask the underlying issues that need addressing in the application logic.

Surfacing Errors:

#### Pros
It allows for immediate debugging and resolution of deadlocks, leading to more robust application designs.

#### Cons
Users may experience frustration or downtime, especially if they are not technically savvy. Additionally, manual intervention is required, which can be time-consuming.

To decide between these options, consider the following:

Frequency of Deadlocks: If deadlocks occur infrequently, it might be more practical to retry transactions automatically. However, frequent deadlocks suggest a deeper issue that needs addressing.
Impact on Critical Transactions: For mission-critical transactions, automatic retries can maintain service availability, but must not introduce performance bottlenecks.
User Experience: In scenarios where the application impacts end-users, surfacing errors may be necessary to ensure they are aware of potential issues and can take corrective actions.

A balanced approach often involves a combination of both methods:

Implement automatic retries for non-critical transactions.
Surface explicit errors for critical transactions or during development phases.

In practice, this could involve setting up an automated system that logs deadlocks and retries non-critical transactions while alerting the support team to investigate critical ones.

**Comparison:**

```
Transaction ID	Is Critical?
1001	No
1002	Yes
1003	No
```

**Code:**

```sql
-- Example of a SQL deadlock detection and resolution mechanism

CREATE OR REPLACE FUNCTION detect_and_resolve_deadlock() RETURNS void AS $$
DECLARE
    deadlock_cycle RECORD;
BEGIN
    -- Attempt to resolve deadlocks by logging them and retrying non-critical transactions.
    FOR deadlock_cycle IN SELECT * FROM get_deadlock_cycle() LOOP
        IF NOT is_critical_transaction(deadlock_cycle.transaction_id) THEN
            -- Log the deadlock and retry the transaction.
            LOG('Deadlock detected, retrying non-critical transaction: ' || deadlock_cycle.transaction_id);
            retry_transaction(deadlock_cycle.transaction_id);
        ELSE
            -- Alert support team for critical transactions.
            RAISE NOTICE 'Critical deadlock detected, notifying support: %', deadlock_cycle.transaction_id;
            notify_support(deadlock_cycle.transaction_id);
        END IF;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

**Example input / output:**

| Transaction ID | Action |
|----------------|--------|
| 1001           | Retry  |
| 1002           | Notify Support |
| 1003           | Retry  |

**Data:**

```json
{
  "transaction_ids": [1001, 1002, 1003],
  "resource_held": {
    "1001": ["lock_a", "lock_b"],
    "1002": ["lock_c", "lock_d"],
    "1003": ["lock_e", "lock_f"]
  },
  "locks_wanted": {
    "1001": ["lock_c", "lock_d"],
    "1002": ["lock_a", "lock_b"],
    "1003": ["lock_a", "lock_e"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Transaction Starts
4. Is Critical?
5. Notify Support
6. Retry Transaction
7. Deadlock Resolved
8. Logged and Retry Attempted

