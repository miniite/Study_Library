---
type: "practice"
difficulty: "Deep"
---

# Give a real-world example of when ON DELETE CASCADE could accidentally delete more data than intended.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Referential Actions](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Referential_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ON DELETE CASCADE is a constraint in relational database management systems that automatically deletes related records from child tables when a record is deleted from its parent table. This can be incredibly useful but also poses risks if not carefully managed, especially in complex schemas.

In this scenario, let's consider a simplified e-commerce application where we have two main entities: Orders and OrderItems. The relationship between these tables involves the Order being the parent and the OrderItems as children. Each order can contain multiple items, but each item is tied to only one order.

#### Example
Schema
SQL
CREATE TABLE Orders (
    id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP NOT NULL
);

CREATE TABLE OrderItems (
    id INT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(id)
        ON DELETE CASCADE -- This is the problematic constraint here.
);
Scenario

Suppose you need to remove an old order that has multiple items in it. You run a simple delete statement:

SQL
DELETE FROM Orders WHERE id = 12345;

Intuitively, this should only affect the Orders table and not anything else because there is no direct foreign key constraint on the OrderItems table. However, due to the ON DELETE CASCADE rule defined in the schema, all associated records in the OrderItems table will also be deleted.

Key Idea

The problem arises when an order has a complex hierarchy or when there are overlapping relationships that weren’t explicitly accounted for in the schema design. For instance, consider another entity called Subscriptions, where each subscription could have multiple items related to its parent order:

SQL
CREATE TABLE Subscriptions (
    id INT PRIMARY KEY,
    user_id INT NOT NULL,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP NOT NULL
);

-- A Subscription can contain many OrderItems.
CREATE TABLE SubscribedOrderItems (
    id INT PRIMARY KEY,
    subscription_id INT NOT NULL,
    order_item_id INT NOT NULL, -- This maps to the ID in OrderItems.
    FOREIGN KEY (subscription_id) REFERENCES Subscriptions(id),
    FOREIGN KEY (order_item_id) REFERENCES OrderItems(id)
        ON DELETE CASCADE
);
Real-World Example

Imagine you are deleting an Order with a subscription that has multiple items. The deletion of the order, via ON DELETE CASCADE, will cascade down to delete all corresponding records in OrderItems, and then those deletions will propagate through SubscribedOrderItems and finally affect Orders. Essentially, the ON DELETE CASCADE constraint might lead to deleting an entire subscription or even a user’s data if not properly managed.

#### Cons
equences

In this case, ON DELETE CASCADE could accidentally delete more than intended. For example:

If you are trying to clean up old orders and their associated items, but there is a subscription tracking those same order items, the deletion of an order might result in unintended consequences.
It can lead to data loss if not tested thoroughly or carefully planned.

#### Solution

To avoid such issues, it's crucial to:

Carefully Design Schemas: Ensure that foreign key relationships are well-defined and do not inadvertently create cascading delete scenarios where they should not exist.
Test Thoroughly: Run test cases with different deletion scenarios before deploying critical changes.
Documentation: Keep detailed documentation of how your schema interacts, especially around foreign key constraints.

#### Closing Summary

ON DELETE CASCADE can be a powerful tool for maintaining data integrity but must be used judiciously to avoid unintended side effects. By understanding the full implications and carefully managing database schemas, you can mitigate risks associated with this constraint.

#### Common mistakes
include not thoroughly testing cascading deletes in complex schemas or assuming that ON DELETE CASCADE will only affect direct parent-child relationships.

**Code:**

```sql
CREATE TABLE Orders (
    id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP NOT NULL
);

CREATE TABLE OrderItems (
    id INT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(id)
        ON DELETE CASCADE
);

CREATE TABLE Subscriptions (
    id INT PRIMARY KEY,
    user_id INT NOT NULL,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP NOT NULL
);

CREATE TABLE SubscribedOrderItems (
    id INT PRIMARY KEY,
    subscription_id INT NOT NULL,
    order_item_id INT NOT NULL, -- This maps to the ID in OrderItems.
    FOREIGN KEY (subscription_id) REFERENCES Subscriptions(id),
    FOREIGN KEY (order_item_id) REFERENCES OrderItems(id)
        ON DELETE CASCADE
);
```

