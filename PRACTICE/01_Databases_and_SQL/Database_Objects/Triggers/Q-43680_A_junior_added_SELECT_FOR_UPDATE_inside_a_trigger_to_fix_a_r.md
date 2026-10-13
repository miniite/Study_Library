---
type: "practice"
difficulty: "Deep"
---

# A junior added SELECT ... FOR UPDATE inside a trigger to fix a race; explain when that's correct and when it just moves the deadlock elsewhere.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Triggers Behavior Performance and Alternatives](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/Triggers_Behavior_Performance_and_Alternatives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, let's first summarize the core issue: a junior developer has introduced SELECT ... FOR UPDATE inside a trigger to fix a race condition. The key points we need to explore are when such an approach is appropriate and when it merely shifts the problem elsewhere.

#### Triaging the Question
Understanding Triggers: Triggers are database objects that execute automatically in response to specific events, typically DML (Data Manipulation Language) statements like INSERT, UPDATE, or DELETE.
Race Conditions: These occur when two transactions attempt to modify the same data simultaneously, leading to unexpected results.
SELECT ... FOR UPDATE: This SQL statement locks selected rows in a table during the transaction until the transaction is committed or rolled back.
Core Solution

The introduction of SELECT ... FOR UPDATE inside a trigger can be correct under specific circumstances but often leads to issues like deadlocks and performance degradation if not used judiciously. Here’s how we should approach this:

Identify the Race Condition: Determine what data is being accessed and modified by different transactions.
Check Transaction Isolation Levels: Understand the isolation level of the transactions involved, as it affects locking behavior.
Ensure Lock Granularity: Use SELECT ... FOR UPDATE to lock only the necessary rows, avoiding excessive locks.

#### First-Principles Solution
When is SELECT ... FOR UPDATE Correct?
Preventing Deadlocks: If multiple transactions need exclusive access to the same data and a race condition is causing deadlocks, adding SELECT ... FOR UPDATE can help by ensuring that one transaction waits until it has exclusive access.
Transaction Isolation Levels: Ensure that your isolation level supports the use of row-level locks effectively.
When Does It Move the Deadlock Elsewhere?
Deadlocking: If the trigger introduces additional transactions that lock the same rows, deadlocks can still occur but at different points in the process.
Performance Impact: Overuse of SELECT ... FOR UPDATE can lead to high lock contention and increased transaction duration, affecting overall system performance.
Detailed Reasoning
Trigger Context: Triggers are inherently part of a larger transaction flow. Adding SELECT ... FOR UPDATE inside a trigger can affect how the trigger interacts with other transactions.
Transaction Flow: Understand the sequence of events that lead to the race condition and ensure that locks are applied at the right points in the transaction.
Lock Granularity: Locking too many rows unnecessarily can lead to contention. Use fine-grained locking to minimize impact.

#### Closing Summary

Adding SELECT ... FOR UPDATE inside a trigger is correct if it helps resolve a race condition by ensuring proper lock management, but it must be done carefully. Overuse or misuse can introduce deadlocks and degrade performance. Always consider the transaction flow and isolation levels when implementing such changes.

#### Common Mistakes/Follow-Ups
Excessive Locking: Using SELECT ... FOR UPDATE on too many rows.
Deadlock Circumvention: Assuming that just because a deadlock occurs in one place, it won’t happen elsewhere due to the added lock.

#### Performance Considerations
Not considering the overall performance impact of increased locking.

#### Example Scenario

Suppose you have a products table with a trigger on UPDATE. If multiple transactions try to update the same product at the same time, adding SELECT ... FOR UPDATE inside the trigger can ensure that one transaction waits until it has exclusive access. However, if another transaction tries to read or write to the same row, it might still lead to deadlocks.

SQL
-- Example of a problematic trigger
CREATE TRIGGER update_product_trigger
AFTER UPDATE ON products
FOR EACH ROW
BEGIN
    SELECT * FROM products WHERE product_id = NEW.product_id FOR UPDATE;
END;
Self-Check

Correctness and completeness confirmed.

