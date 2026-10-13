---
type: "practice"
difficulty: "Intermediate"
---

# Explain how a foreign key with ON DELETE SET NULL interacts with a NOT NULL constraint on the same column, and how the DB resolves the conflict.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Foreign Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Foreign_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a database design where a foreign key (FK) is defined with an ON DELETE SET NULL action in a table that has a NOT NULL constraint on the same column. This interaction can lead to interesting and sometimes unexpected behaviors. Let's break down how this works from first principles.

First, let's understand what each of these constraints does:

Foreign Key (FK): Ensures referential integrity by linking columns in one table with columns in another.
ON DELETE SET NULL: When a referenced row is deleted, the values in the referencing column(s) are set to NULL.
NOT NULL Constraint: A column cannot contain any null value; it must always have a valid entry.

Now, let's consider how these interact. The key point here is that both constraints are trying to enforce different rules on the same column. If a row in the parent table gets deleted, and there's an associated foreign key with ON DELETE SET NULL, those values will be set to NULL. However, if that column has a NOT NULL constraint, setting it to NULL would violate this rule.

To resolve this conflict, the database system must decide how to handle the situation. Here’s how it typically works in most SQL databases:

Violation and Rollback: The database will attempt to set the foreign key values to NULL. If a row with these foreign keys has a NOT NULL constraint, the transaction will fail. This means that deleting the referenced row in the parent table cannot proceed without first addressing this issue.

Manual Resolution: One way to resolve this is by modifying the rows in the child table where the foreign key values are NULL and setting them to appropriate non-null values manually before attempting the delete on the parent table.

Design Considerations: To avoid such conflicts, it’s important to carefully design your schema. For instance, you might want to ensure that deleting a row from the parent does not leave orphaned rows in the child table by using cascading deletes or other design patterns.

In summary, when ON DELETE SET NULL and NOT NULL constraints conflict on the same foreign key column, the database will attempt to set the values to NULL, but this may fail if the NOT NULL constraint is violated. The resolution usually requires manual intervention or rethinking the schema design.

**Comparison:**

```
Parent	Child
id	name
1	A
2	B
```

**Code:**

```sql
-- Example of setting up a foreign key with ON DELETE SET NULL and NOT NULL constraints
CREATE TABLE Parent (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE Child (
    parent_id INT,
    -- NOT NULL constraint on the foreign key column
    CONSTRAINT fk_child_parent FOREIGN KEY (parent_id) REFERENCES Parent(id) ON DELETE SET NULL,
    CONSTRAINT chk_child_parent NOT NULL CHECK (parent_id IS NOT NULL)
);
```

**Example input / output:**

```

Given the schema above:

If we try to delete Parent A (id=1), the following will happen:
SQL
Copy
DELETE FROM Parent WHERE id = 1; -- This will fail because of NOT NULL constraint in Child table.
```

**Data:**

```json
{
  "Parent": [
    { "id": 1, "name": "Parent A" },
    { "id": 2, "name": "Parent B" }
  ],
  "Child": [
    { "parent_id": 1 },
    { "parent_id": 2 }
  ]
}
```

**Flow:**

1. Parent
2. Set Child.parent_id to NULL
3. Delete Parent?
4. Check NOT NULL constraint inChild
5. Fails, Rollback Delete
6. Manually set values orre-design schema

