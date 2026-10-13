---
type: "practice"
difficulty: "Intermediate"
---

# Can you name a constraint explicitly and why would you want to name your constraints?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Normal Forms](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In database management systems (DBMS), constraints are rules that ensure the accuracy, consistency, and integrity of data within tables. Naming these constraints is a best practice that helps in maintaining code readability and ease of maintenance.

#### Triage

The question asks us to name an explicit constraint and explain why it would be useful to name such constraints. Constraints can include primary keys, foreign keys, unique constraints, check constraints, etc. The focus here should be on understanding the importance of naming them for effective database management.

#### First-Principles Solution
Why Name Constraints?
Clarity and Readability: Naming constraints makes it easier to understand the purpose and functionality of each constraint within a schema.
Maintainability: Properly named constraints can help when troubleshooting or making changes, as they provide clear context about their role in the database.
Documentation: Constraints are part of the documentation for your database, and naming them ensures that the document is more comprehensible.

#### Example
Naming a Primary Key

#### Cons
ider a scenario where you have an orders table with columns like order_id, customer_id, product_id, etc. A primary key constraint can be applied to the order_id column.

SQL
ALTER TABLE orders ADD CONSTRAINT order_id_pk PRIMARY KEY (order_id);

#### Example
Naming a Foreign Key

Now, let's say you have an orders_items table that references both orders and products. You might want to establish a foreign key constraint:

SQL
ALTER TABLE orders_items
ADD CONSTRAINT fk_order_id FOREIGN KEY (order_id) REFERENCES orders(order_id),
ADD CONSTRAINT fk_product_id FOREIGN KEY (product_id) REFERENCES products(product_id);

#### Closing Summary
+ Common Mistakes/Follow-Ups

Naming constraints is crucial for maintainability and documentation. It's a good practice to follow naming conventions that make sense in the context of your application or business requirements.

#### Common mistakes
include:

Forgetting to name constraints, leading to unclear code.
Using inconsistent or ambiguous names, making it hard to understand their purpose.
Triangulating with a Scenario

Imagine you're working on an e-commerce application where users can place multiple orders. By naming the foreign key constraint order_id_fk and the primary key constraint order_id_pk, you ensure that anyone reading your code knows exactly what these constraints are for, without needing to refer to the schema documentation.

#### Common Mistakes
Not naming any constraints at all.
Using generic names like pk1, fk1, which provide no context.
Overcomplicating names unnecessarily.

**Comparison:**

| order_id | customer_id |
| --- | --- |
| 1       | 10         |
| 2       | 11         |
| product_id | name        |
| --- | --- |
| 100       | Product A   |
| 101       | Product B   |

**Code:**

```sql
-- Example of adding primary and foreign key constraints
ALTER TABLE orders ADD CONSTRAINT order_id_pk PRIMARY KEY (order_id);

ALTER TABLE orders_items
ADD CONSTRAINT fk_order_id FOREIGN KEY (order_id) REFERENCES orders(order_id),
ADD CONSTRAINT fk_product_id FOREIGN KEY (product_id) REFERENCES products(product_id);
```

**Example input / output:**

```text
-- Adding constraints to the orders and orders_items tables

-- Orders table
ALTER TABLE orders ADD CONSTRAINT order_id_pk PRIMARY KEY (order_id);

-- Orders_items table
ALTER TABLE orders_items
ADD CONSTRAINT fk_order_id FOREIGN KEY (order_id) REFERENCES orders(order_id),
ADD CONSTRAINT fk_product_id FOREIGN KEY (product_id) REFERENCES products(product_id);
```

**Data:**

```json
{
  "orders": [
    { "order_id": 1, "customer_id": 10 },
    { "order_id": 2, "customer_id": 11 }
  ],
  "products": [
    { "product_id": 100, "name": "Product A" },
    { "product_id": 101, "name": "Product B" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Create Orders Table
6. Add Constraints?
7. Name Primary Key Constraint
8. Name Foreign KeyConstraint(s)
9. Continue Without Naming
10. Create Orders_Items Table
11. Add Constraints?
12. Name Foreign KeyConstraint(s)
13. Document Changes
14. Continue Without Naming

