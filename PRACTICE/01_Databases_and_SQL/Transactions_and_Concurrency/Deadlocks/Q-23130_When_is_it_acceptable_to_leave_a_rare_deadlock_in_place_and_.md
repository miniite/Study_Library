---
type: "practice"
difficulty: "Intermediate"
---

# When is it acceptable to leave a rare deadlock in place and simply rely on retry, and how do you communicate that decision?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Retry Strategy](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Retry_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle deadlocks, one common approach is to rely on retry mechanisms rather than actively detecting and resolving them, especially when deadlocks are infrequent. This strategy balances between performance and complexity, as it avoids the overhead of deadlock detection systems. However, understanding the context and potential risks is crucial before making such a decision.

#### Triage

The question asks about scenarios where you might choose to accept rare deadlocks in favor of simpler retry mechanisms rather than implementing more complex deadlock resolution techniques. The interviewer is probing your understanding of how deadlocks can impact system performance, the trade-offs between various strategies, and your communication skills regarding decisions made.

We assume that:

Deadlock detection introduces additional complexity and overhead.
Deadlocks are rare but may still occur in certain scenarios.
Relying on retries has a cost in terms of increased database transactions and potential application-level logic adjustments.

#### Solution

Deadlocks can arise when two or more transactions hold locks on different resources while waiting for each other to release those locks. In some cases, it might be acceptable to leave these deadlocks unresolved if the frequency is low enough not to significantly impact system performance.

Step-by-Step Reasoning

Understanding Deadlock Frequency and Impact:

Assess how often deadlocks occur in your application.

#### Cons
ider the criticality of the transactions involved; if they are non-critical, a slight delay might be acceptable.
Evaluate whether the impact on overall system performance is minimal.

Implementing Retry Mechanisms:

Introduce retry logic at the transaction level to automatically attempt re-execution upon detecting deadlocks.
This can be done using database features like ON CONFLICT in SQL databases or transaction retries via application-level logic.

Monitoring and Logging Deadlock Events:

Continuously monitor deadlock events to ensure they remain infrequent.
Log detailed information about each deadlock event for post-mortem analysis and to identify patterns that might indicate the need for improvement.

Communicating the Decision:

Clearly document why you chose to rely on retry mechanisms over more complex resolution techniques.
Highlight the potential risks, such as increased transaction latency, in your communication with stakeholders.
Provide a plan for revisiting this decision if deadlock frequency increases significantly.

#### Example Scenario
s
Low-Criticality Transactions: Deadlocks involving read-only or low-critical transactions are less problematic and can be tolerated more easily.
Read-Heavy Workloads: In systems where write operations are infrequent, deadlocks might not cause significant performance issues and can be managed with retries.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, relying on retry mechanisms for rare deadlocks is a viable strategy when the impact of deadlocks is minimal. However, it's crucial to continuously monitor the system and document any risks associated with this approach.

#### Common Mistakes

Overlooking the need for thorough monitoring and logging.
Ignoring the potential for increased transaction latency.
Failing to communicate effectively with stakeholders about the decision and its implications.

#### Follow-Ups

Discuss specific tools or techniques used for deadlock detection and management.
Explore alternative strategies, such as lock ordering policies, which could be more effective in certain scenarios.

#### Cons
ider implementing a hybrid approach that combines retry mechanisms with basic deadlock prevention measures.

**Comparison:**

```
Transaction ID	Status	Timestamp
TX1	Retry	2023-10-01T14:00:00Z
TX2	Success	2023-10-05T16:00:00Z
Deadlock Event	Involved Transactions	Impact
Yes	TX1, TX2	Minor
No		Not Applicable
```

**Code:**

```sql
-- Example SQL code to implement retry logic using ON CONFLICT
BEGIN;
SELECT * FROM orders WHERE order_id = 123 FOR UPDATE;
-- Simulate another transaction holding a conflicting lock
SAVEPOINT my_savepoint;
UPDATE orders SET status = 'processed' WHERE order_id = 123;
IF EXISTS (SELECT * FROM orders WHERE order_id = 456 FOR UPDATE) THEN
    ROLLBACK TO SAVEPOINT my_savepoint;
    -- Retry logic here, possibly with a delay or backoff strategy
END IF;
COMMIT;
```

**Example input / output:**

```text
Input:
BEGIN;
SELECT * FROM orders WHERE order_id = 123 FOR UPDATE;

Output:
Deadlock detected. Retrying transaction...

New Input (after retry):
BEGIN;
SELECT * FROM orders WHERE order_id = 123 FOR UPDATE;

New Output:
Transaction completed successfully.
```

**Data:**

```json
{
  "transaction_logs": [
    {
      "timestamp": "2023-10-01T14:00:00Z",
      "deadlock_event": true,
      "involved_transactions": ["TX1", "TX2"],
      "impact": "minor"
    },
    {
      "timestamp": "2023-10-05T16:00:00Z",
      "deadlock_event": false
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Are deadlocks frequent?
5. Monitor and log
6. Rethink strategy
7. Document decision
8. Implement deadlock resolution

