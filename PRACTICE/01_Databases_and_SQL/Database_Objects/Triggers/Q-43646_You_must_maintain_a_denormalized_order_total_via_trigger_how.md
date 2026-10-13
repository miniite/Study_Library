---
type: "practice"
difficulty: "Expert"
---

# You must maintain a denormalized order_total via trigger; how do you keep it correct under concurrent line-item inserts without serializing all writes?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Triggers Behavior Performance and Alternatives](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/Triggers_Behavior_Performance_and_Alternatives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a denormalized order_total remains accurate under concurrent line-item inserts, we need to address the challenge of maintaining data integrity in a distributed or multi-threaded environment where updates can overlap. The key is to design a trigger mechanism that can handle concurrent writes without causing race conditions.

#### Triage
Understanding the Problem

The question asks how to keep an order_total denormalized field correct under concurrent line-item inserts, specifically without serializing all writes. This means we need to implement a solution that allows multiple transactions to update the total simultaneously but ensures it remains consistent.

#### Assumptions
and Key Ideas
Concurrency: Multiple line-item insertions can occur concurrently.
Denormalized Data: The order_total field is not part of the main order table and needs to be updated after each insertion.
Trigger Mechanism: We will use a database trigger to handle the updates.

#### First-Principles Solution

To maintain consistency, we can use a combination of a trigger and a locking mechanism or an atomic update approach. Here’s how:

Use a Row-Level Locking Mechanism:
A row-level lock is placed on the orders table to prevent concurrent updates from overwriting each other.
Atomic Update: Use an UPDATE statement within the trigger that performs an atomic operation.
Step-by-Step Solution

Create a Trigger:

SQL
CREATE TRIGGER update_order_total AFTER INSERT ON order_items FOR EACH ROW
BEGIN
    UPDATE orders o
    SET o.order_total = (SELECT SUM(item_price * quantity) FROM order_items WHERE order_id = NEW.order_id)
    WHERE o.id = NEW.order_id;
END;

Handling Concurrency:

Ensure that the orders table has a lock placed on it when the trigger is fired.
Use an UPDATE statement to sum up the new line-item values and update the order_total.

Testing with Sample Data:

SQL
-- Sample data for orders and order_items
CREATE TABLE orders (
    id INT PRIMARY KEY,
    order_total DECIMAL(10, 2)
);

INSERT INTO orders (id, order_total) VALUES (1, 0);

CREATE TABLE order_items (
    id INT PRIMARY KEY,
    order_id INT,
    item_price DECIMAL(10, 2),
    quantity INT
);

-- Concurrent inserts
START TRANSACTION;
INSERT INTO order_items (order_id, item_price, quantity) VALUES (1, 5.99, 2);
COMMIT;

START TRANSACTION;
INSERT INTO order_items (order_id, item_price, quantity) VALUES (1, 3.99, 1);
COMMIT;

Expected Outcome:

The order_total for order ID 1 should be updated to (5.99 \times 2 + 3.99 \times 1 = 15.97).

#### Closing Summary
and Common Mistakes
Using a single trigger can lead to performance issues if there are many concurrent transactions.
Ensure that the UPDATE operation inside the trigger is atomic.

#### Cons
ider using optimistic locking or other concurrency control mechanisms depending on your database system.

#### Common Mistakes/Follow-Ups
Race Conditions: Ensure no two triggers run at the same time for the same order.
Performance Overhead: Triggers can introduce significant overhead. Evaluate the need and impact of the solution.
Concurrency Control: Understand how your database handles locks and transactions in concurrent scenarios.

