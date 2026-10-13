---
type: "practice"
difficulty: "Basic"
---

# What is a foreign key and what does referential integrity mean?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Lock Granularity](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

Two-part basic-concept question: (1) what is a foreign key, and (2) what does referential integrity mean? These two ideas are inseparable — a foreign key is the mechanism, and referential integrity is the guarantee the mechanism enforces. The interviewer is checking that you understand how relational databases link tables together and how the database keeps those links from ever becoming "dangling" (pointing at rows that don't exist). A strong answer defines both crisply, shows a two-table example, and then explains what happens on DELETE/UPDATE (the ON DELETE/ON UPDATE referential actions), because that is where referential integrity becomes concrete.

Assumption: we are talking about a relational database (MySQL, PostgreSQL, etc.) with the constraint actually enforced by the engine (e.g., InnoDB, not MyISAM). Plan: define foreign key → define referential integrity → show the enforcement rules on insert/update/delete → cover the referential actions → summary and follow-ups.

What is a foreign key?

A foreign key (FK) is a column (or set of columns) in one table whose value must match a value of the primary key (or a unique key) in another table — it is a reference from a child table to a parent table. It is how relational databases represent relationships.

Classic example: an orders table has a customer_id column that is a foreign key referencing customers(customer_id). This says: "every order must belong to a real customer." The customers table is the parent (referenced) table; orders is the child (referencing) table.

So a foreign key is not a new kind of stored value — it is a constraint that ties the two tables together and tells the database to police that relationship.

What does referential integrity mean?

Referential integrity is the guarantee that every foreign key value points to an existing row in the parent table — there are no "orphan" or "dangling" references. In our example, referential integrity means: you can never have an order whose customer_id refers to a customer that does not exist. The relationship is always valid.

The database enforces this automatically at two moments:

On INSERT/UPDATE of the child. If you try to insert an order with customer_id = 999 but no customer 999 exists, the DBMS rejects the operation. (Exception: a foreign-key column that allows NULL may be NULL, meaning "no relationship yet" — a NULL FK does not violate integrity.)

On DELETE/UPDATE of the parent. If you try to delete customer 5 while orders still reference customer 5, that would leave orphans. What happens next is controlled by the referential action you defined:

RESTRICT / NO ACTION (the safe default): the delete/update is blocked while child rows exist. You must remove or reassign the orders first.
CASCADE: deleting the customer also deletes all their orders automatically (and ON UPDATE CASCADE propagates a key change down to the children).
SET NULL: the child's customer_id is set to NULL (the FK column must be nullable), meaning "customer removed, order kept but unlinked."
SET DEFAULT: the child's FK is set to a predefined default value (supported by some engines).

These actions are the practical heart of referential integrity: they define exactly how the database keeps references valid when the parent changes.

Why it matters

Without referential integrity you get corrupt, meaningless data: orders for customers who vanished, line items for orders that were deleted, employees pointing to a department that no longer exists. Reports break, joins return garbage, and application code has to defensively check for missing parents everywhere. By declaring foreign keys, you push that guarantee down into the database, where it is enforced consistently no matter which application or script writes the data. It is a core reason we use an RDBMS rather than plain files.

A concrete walk-through

Given customers(customer_id PK) and orders(order_id PK, customer_id FK → customers):

Insert order for existing customer 1 → allowed.
Insert order for non-existent customer 999 → rejected (would break referential integrity).
Delete customer 1 who has orders, with ON DELETE RESTRICT → blocked.
Delete customer 1 who has orders, with ON DELETE CASCADE → customer and their orders deleted.
Delete customer 1 who has orders, with ON DELETE SET NULL → customer deleted, their orders kept with customer_id = NULL.

That single trace demonstrates both concepts working together.

Summary / what a strong answer emphasizes

Say it in two lines: a foreign key is a column in a child table that must match a primary/unique key in a parent table, and referential integrity is the guarantee that it always does — no orphan references. Then earn extra credit by explaining what happens when the parent is deleted or updated (RESTRICT / CASCADE / SET NULL), since that is where interviewers see whether you actually understand enforcement versus just memorizing a definition.

#### Common mistakes
thinking a foreign key must reference a primary key only (it can reference any UNIQUE column); forgetting that a nullable FK is allowed to be NULL; and assuming CASCADE is the default (the default is usually RESTRICT/NO ACTION). Also, in MySQL, foreign keys are only enforced by transactional engines like InnoDB — MyISAM parses but ignores them.

Likely follow-ups: "What is the difference between a primary key and a foreign key?" (PK = unique identity within its own table and cannot be NULL; FK = a reference to another table's key and can be NULL and can repeat), "Can a table reference itself?" (yes — a self-referencing FK, e.g., employee.manager_id → employee.employee_id), and "What are the downsides of foreign keys?" (extra write-time checks/locks can add overhead at very high scale, which is why some large systems enforce relationships in the application layer instead).

**Comparison:**

```
ON DELETE / ON UPDATE action	Behavior when parent row is deleted/updated	Requires nullable FK?
RESTRICT / NO ACTION (default)	Operation blocked while child rows exist	No
CASCADE	Child rows are deleted / key change propagated to children	No
SET NULL	Child FK set to NULL ("unlinked")	Yes
SET DEFAULT	Child FK set to its default value	No (default must exist)
Concept	Foreign Key	Referential Integrity
What it is	A column referencing another table's key	The guarantee that the reference is always valid
Role	The mechanism/relationship	The rule the mechanism enforces
Can be NULL	Yes (means "no relationship")	NULL FK does not violate it
```

**Code:**

```sql
-- Parent (referenced) table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name        VARCHAR(100) NOT NULL
);

-- Child (referencing) table with a foreign key + referential action
CREATE TABLE orders (
    order_id    INT PRIMARY KEY,
    customer_id INT,                     -- nullable FK => "no customer yet" is allowed
    amount      DECIMAL(10,2) NOT NULL,
    CONSTRAINT fk_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
        ON DELETE RESTRICT               -- block deleting a customer who still has orders
        ON UPDATE CASCADE                -- if customer_id changes, propagate to orders
);
```

**Example input / output:**

```text
customers: (1, 'Alice'), (2, 'Bob')

INSERT INTO orders VALUES (100, 1, 500.00);   -- OK: customer 1 exists
INSERT INTO orders VALUES (101, 999, 20.00);  -- ERROR: no customer 999 (referential integrity)
INSERT INTO orders VALUES (102, NULL, 15.00); -- OK: nullable FK, "no customer yet"

-- With ON DELETE RESTRICT:
DELETE FROM customers WHERE customer_id = 1;  -- ERROR: order 100 still references customer 1

-- If FK were ON DELETE CASCADE instead:
DELETE FROM customers WHERE customer_id = 1;  -- OK: deletes customer 1 AND order 100

-- If FK were ON DELETE SET NULL instead:
DELETE FROM customers WHERE customer_id = 1;  -- OK: order 100 kept, its customer_id set to NULL
```

**Flow:**

1. orders - child
2. customers - parent
3. must match
4. Yes
5. No
6. RESTRICT
7. CASCADE
8. SET NULL
9. customer_id PK
10. customer_id FK
11. Write to childor delete/update parent
12. FK still points toa valid parent row?
13. Allowed
14. Referential action
15. Reject/Block
16. Delete/update children too
17. Child FK becomes NULL

