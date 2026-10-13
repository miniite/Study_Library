---
type: "practice"
difficulty: "Intermediate"
---

# What happens to referential integrity if you delete a row that is referenced by a foreign key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Polymorphic Associations](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deleting a row in a database, the behavior regarding referential integrity depends on how foreign keys are defined. Specifically, if a row being deleted has another row referencing it via a foreign key constraint, the database must ensure that this relationship is respected to maintain data consistency and avoid orphaned records or cascading deletions.

To understand the impact, let's break down the scenario step-by-step:

#### Overview
When you delete a row in your primary table (let’s call it ParentTable), the row from the secondary table (let’s call it ChildTable) that references this row via a foreign key (ChildTable.ParentID REFERENCES ParentTable.ID) is considered an orphan if no corresponding row exists in ParentTable. The database's behavior towards such a deletion can be controlled by defining different actions on delete for the foreign key constraint.

#### Triaging the Question
The interviewer wants to explore how you handle referential integrity constraints, which are critical for maintaining data consistency and preventing anomalies like orphaned records or unintended cascading deletions.

#### First-Principles Solution

No Action (CASCADE): If a foreign key is defined with ON DELETE CASCADE, the referenced row in the secondary table (ChildTable) will also be deleted when you delete the parent row in the primary table (ParentTable).
Restrict: If the constraint is set to ON DELETE RESTRICT, deleting the parent row will result in an error and prevent the deletion from occurring, as it would leave orphaned records.
Set Null (SET NULL): With ON DELETE SET NULL, the foreign key field in the child table will be automatically set to null when you delete the corresponding row in the parent table. This can help avoid orphaned records but might lead to inconsistencies if not properly managed.

#### Closing Summary
and Common Mistakes:

The key takeaway is understanding that referential integrity constraints must be carefully considered during schema design.
Forgetting or misconfiguring foreign keys can lead to data anomalies like orphaned records, which violate the integrity of your database.
A common mistake is not defining appropriate actions on delete for foreign keys, leading to inconsistent behavior when rows are deleted.

**Code:**

```sql
-- Example schema with foreign key constraints

CREATE TABLE ParentTable (
    ID INT PRIMARY KEY,
    Name VARCHAR(255)
);

CREATE TABLE ChildTable (
    ID INT PRIMARY KEY,
    ParentID INT,
    FOREIGN KEY (ParentID) REFERENCES ParentTable(ID) ON DELETE CASCADE
);
```

**Example input / output:**

```text
Input:
DELETE FROM ParentTable WHERE ID = 1;

Output (with ON DELETE CASCADE):
ParentTable: [ { "ID": 2, "Name": "Parent2" } ]
ChildTable: [ ]

Explanation:
Deleting the row with ID=1 from ParentTable cascaded to delete both referencing rows in ChildTable.
```

**Data:**

```json
{
  "ParentTable": [
    {"ID": 1, "Name": "Parent1"},
    {"ID": 2, "Name": "Parent2"}
  ],
  "ChildTable": [
    {"ID": 1, "ParentID": 1},
    {"ID": 2, "ParentID": 1}
  ]
}
```

