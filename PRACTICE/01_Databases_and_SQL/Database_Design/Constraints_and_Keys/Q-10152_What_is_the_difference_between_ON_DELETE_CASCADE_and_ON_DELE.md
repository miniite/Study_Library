---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between ON DELETE CASCADE and ON DELETE SET NULL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Referential Actions](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Referential_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between ON DELETE CASCADE and ON DELETE SET NULL, we need to delve into how these constraints affect the relationships between tables in a database. Essentially, both are mechanisms used to maintain referential integrity when a row is deleted from one table that has related data in another.

#### Triaging the Question

The question probes your understanding of database normalization and referential integrity. Specifically:

Referential Integrity: Ensuring that relationships between tables are maintained.
ON DELETE CASCADE: What happens to related rows in other tables when a row is deleted from a parent table.
ON DELETE SET NULL: The action taken on related rows when a primary key or foreign key is deleted.

#### Assumptions
We have two tables: ParentTable and ChildTable.
There is a relationship between these tables through a foreign key in ChildTable referencing the primary key of ParentTable.

#### First-Principles Solution
ON DELETE CASCADE

When you apply ON DELETE CASCADE, it means that if a row from the parent table (ParentTable) is deleted, all rows in the child table (ChildTable) that reference this parent row will also be automatically deleted. This helps maintain referential integrity by ensuring that there are no orphaned records.

#### Example
Suppose we have the following schema:

SQL
CREATE TABLE ParentTable (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE ChildTable (
    id INT PRIMARY KEY,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES ParentTable(id) ON DELETE CASCADE
);

If we delete a row from ParentTable:

SQL
DELETE FROM ParentTable WHERE id = 1;

All rows in ChildTable that reference this id will also be deleted.

ON DELETE SET NULL

In contrast, ON DELETE SET NULL means that when a row is deleted from the parent table, the foreign key value in the child table(s) that references it will be set to NULL. This ensures that there are no orphaned records but can lead to null values being present in your database.

#### Example
Using the same schema:

SQL
CREATE TABLE ChildTable (
    id INT PRIMARY KEY,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES ParentTable(id) ON DELETE SET NULL
);

If we delete a row from ParentTable:

SQL
DELETE FROM ParentTable WHERE id = 1;

All rows in ChildTable that reference this id will have their parent_id set to NULL.

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake 1: Confusing ON DELETE CASCADE with ON DELETE SET NULL. Both are used for referential integrity, but the actions they enforce when a parent row is deleted differ significantly.
Mistake 2: Ignoring potential performance implications. ON DELETE CASCADE can lead to cascading deletes, which might be too aggressive in some scenarios.
Common Follow-Ups

What are the trade-offs between these two constraints?

ON DELETE CASCADE ensures referential integrity but can cause excessive data deletion.
ON DELETE SET NULL maintains referential integrity by allowing nulls but can lead to orphaned records.

When would you use one over the other?

Use ON DELETE CASCADE when you want to ensure that deleting a parent record will always cascade down and delete related child records.
Use ON DELETE SET NULL when you need referential integrity but want to avoid excessive data deletion, or when having null values is acceptable.

Are there any other constraints or actions we can apply?

You can also use ON UPDATE CASCADE, which ensures that if a parent row's primary key is updated, all child rows' foreign keys will be updated accordingly.

#### Cons
ider using cascading updates (ON UPDATE CASCADE) in conjunction with deletions.

**Code:**

```

-- Example schema for ON DELETE CASCADE CREATE TABLE ParentTable ( id INT PRIMARY KEY, name VARCHAR(255) );

CREATE TABLE ChildTable ( id INT PRIMARY KEY, parent_id INT, FOREIGN KEY (parent_id) REFERENCES ParentTable(id) ON DELETE CASCADE );

CODE
Copy

```sql
-- Example schema for ON DELETE SET NULL
CREATE TABLE ParentTable (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE ChildTable (
    id INT PRIMARY KEY,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES ParentTable(id) ON DELETE SET NULL
);
```

