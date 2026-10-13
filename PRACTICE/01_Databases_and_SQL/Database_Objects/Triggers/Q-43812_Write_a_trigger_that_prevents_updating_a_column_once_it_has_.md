---
type: "practice"
difficulty: "Intermediate"
---

# Write a trigger that prevents updating a column once it has been set to a final value.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Triggers Behavior Performance and Alternatives](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/Triggers_Behavior_Performance_and_Alternatives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a trigger that prevents updating a column once it has been set to a final value, we need to ensure that any update operation on the table is checked before it gets executed. If the column in question (let's call it status) has already been set to its "final" value, the update should be blocked.

#### Triage

The question requires us to implement a database trigger mechanism that prevents updates to a specific column (status), if it has reached a particular final state. The key points we need to consider include:

Identifying the Final State: Determine what constitutes the "final" value for the status column.
Checking Existing Value: Before an update, check if the existing value of the status column is already set to the final state.
Blocking Updates: If the condition is met (i.e., the status has reached its final value), block or ignore the update operation.
Step-by-Step Solution

Assuming we have a table named orders with columns order_id, customer_name, and status. Let’s assume the "final" state of the status column is 'Completed'.

Identify the Condition: We need to check if the current value of status is already set to 'Completed'.
Trigger Implementation:
The trigger should be placed on the table where the update operation will take place.
Use a BEFORE UPDATE trigger to intercept the update before it commits.

#### First-Principles Solution
SQL
CREATE TRIGGER prevent_completed_update
BEFORE UPDATE ON orders
FOR EACH ROW
BEGIN
    IF NEW.status = 'Completed' THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Cannot change a completed order status.';
    END IF;
END;
Explanation
Trigger Name and Table: The trigger is named prevent_completed_update and is placed on the orders table.
FOR EACH ROW: This ensures that the trigger logic runs for each row being updated.
Condition Check: The condition checks if the new value of the status column (NEW.status) is 'Completed'.
SIGNAL SQLSTATE: If the condition is met, a signal is raised to block the update operation.

#### Closing Summary

This trigger ensures that once an order's status has been marked as "Completed," no further updates can change it, thus preventing any accidental or unauthorized modifications to completed orders.

#### Common Mistakes
and Follow-Ups
Incorrect Condition: Ensure the condition correctly identifies the final state.
Ignoring Error Handling: Make sure proper error handling is in place when raising a signal.

#### Performance Considerations
Be mindful of performance implications, as triggers can impact transactional performance. Consider indexing if necessary.

