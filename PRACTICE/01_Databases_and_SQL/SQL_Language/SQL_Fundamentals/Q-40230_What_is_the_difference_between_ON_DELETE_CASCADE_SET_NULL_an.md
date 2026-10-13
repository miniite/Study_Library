---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between ON DELETE CASCADE, SET NULL, and RESTRICT for a foreign key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Foreign Keys and Referential Integrity](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with foreign keys in SQL databases, understanding how to manage referential integrity during deletions can be crucial. Specifically, ON DELETE CASCADE, SET NULL, and RESTRICT are three common options that determine what happens when a referenced row is deleted from the parent table. Let's break down each of these constraints step by step.

#### Triage

The question asks for an explanation of how different cascading actions (CASCADE, SET NULL, RESTRICT) affect foreign key relationships during deletion. The interviewer may be probing your understanding of SQL syntax, referential integrity, and the potential implications on data consistency and application logic.

#### First-Principles Solution

ON DELETE CASCADE:

When a row in the parent table is deleted, all rows in the child table that reference this row are also deleted.
This option ensures that no orphaned records remain in the child table.

#### Example
If you delete an employee from the employees table (parent), then any corresponding entries in the orders table (child) would be automatically deleted.

SET NULL:

When a row in the parent table is deleted, the foreign key values in all related rows of the child table are set to NULL.
This option allows orphaned records but maintains referential integrity by not allowing nulls on the parent side.

#### Example
If you delete an employee from the employees table (parent), then their corresponding entries in the orders table (child) would have their employee_id field set to NULL.

RESTRICT:

This is a default constraint that prevents deleting rows if they are referenced by foreign key constraints.
It enforces referential integrity, making sure that no deletions can occur on the parent table that would leave orphaned records in the child table.

#### Example
If you attempt to delete an employee from the employees table (parent) who has orders recorded in the orders table (child), the database will prevent this operation.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, ON DELETE CASCADE is useful when you want to ensure that related data is automatically cleaned up, but it can lead to orphaned records if not used carefully. SET NULL allows for flexibility by setting foreign key values to null, which can be useful in certain scenarios where orphaned records are acceptable. On the other hand, RESTRICT enforces strong referential integrity at the cost of preventing certain deletions.

#### Common mistakes
include:

Overusing CASCADE without considering potential orphaned records.
Failing to understand the implications of allowing NULL values on foreign keys.
Ignoring the default RESTRICT behavior and its importance in maintaining data consistency.

#### Follow-ups
could involve discussing scenarios where each constraint might be more suitable, or delving into performance considerations and database normalization best practices.

**Comparison:**

| Employees | Orders        |
|-----------|---------------|
| id | name         | order_id | employee_id |
| 1  | Alice        | 1        | 1           |
| 2  | Bob          | 2        | 2           |
| 3  | Charlie      | 3        | 3           |
|----------|-------------|
| After deleting employee id=2: | After deleting employee id=2: |
| Employees table: { "id": 1, "name": "Alice" }, { "id": 3, "name": "Charlie" } | Employees table: { "id": 1, "name": "Alice" }, { "id": 3, "name": "Charlie" } |
| Orders table: {} | Orders table: [{ "order_id": 2, "employee_id": null }] |

**Code:**

```sql
-- Example of ON DELETE CASCADE
ALTER TABLE orders
ADD CONSTRAINT fk_employee_id
FOREIGN KEY (employee_id) REFERENCES employees(id)
ON DELETE CASCADE;

-- Example of SET NULL
ALTER TABLE orders
ADD CONSTRAINT fk_employee_id
FOREIGN KEY (employee_id) REFERENCES employees(id)
ON DELETE SET NULL;

-- Example of RESTRICT
ALTER TABLE orders
ADD CONSTRAINT fk_employee_id
FOREIGN KEY (employee_id) REFERENCES employees(id);
```

**Example input / output:**

```text
Input:
DELETE FROM employees WHERE id = 2;

Output (CASCADE):
Employees table: { "id": 1, "name": "Alice" }, { "id": 3, "name": "Charlie" }
Orders table: {}

Output (SET NULL):
Employees table: { "id": 1, "name": "Alice" }, { "id": 3, "name": "Charlie" }
Orders table: [{ "order_id": 2, "employee_id": null }]
```

**Data:**

```json
{
  "employees": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" },
    { "id": 3, "name": "Charlie" }
  ],
  "orders": [
    { "order_id": 1, "employee_id": 1 },
    { "order_id": 2, "employee_id": 2 },
    { "order_id": 3, "employee_id": 3 }
  ]
}
```

**Flow:**

1. Set Null
2. Delete Employee 2
3. Is ON DELETE SET NULL?
4. Set Orders of Employee 2 toNULL
5. Employees: { 'id': 1, 'name':'Alice' }, { 'id': 3, 'name':'Charlie' }
6. Orders: [{ 'order_id': 2,'employee_id': null }]
7. Cascade
8. Delete Employee 2
9. Is ON DELETE CASCADE?
10. Cascade Delete Orders ofEmployee 2
11. Employees: { 'id': 1, 'name':'Alice' }, { 'id': 3, 'name':'Charlie' }
12. Orders: {}
13. Parent Table (Employee)
14. Child Table (Order)

