---
type: "practice"
difficulty: "Basic"
---

# What does the ALTER TABLE statement do and give three things you can change with it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Schema Evolution in Lakehouse Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The ALTER TABLE statement in SQL is used to modify an existing table in a database. Essentially, it allows you to add, modify, or delete columns, constraints, and other properties of a table without having to recreate the entire table from scratch. Here's what you can change with ALTER TABLE:

Adding Columns: You can add new columns to your table.
Modifying Columns: You can alter existing columns (e.g., changing their data type or constraints).
Deleting Columns: You can remove unnecessary columns.

Let’s dive into each of these in more detail.

#### Triage

The ALTER TABLE statement is fundamentally about making changes to the structure and integrity of an existing table. The key points are:

What changes can be made (adding, modifying, or deleting columns).
The syntax for each type of change.
Potential impacts on data integrity and performance.

#### First-Principles Solution

Adding Columns:

SQL
ALTER TABLE table_name ADD column_name datatype [column_constraint];

For example:

SQL
ALTER TABLE employees ADD department_id INTEGER;

Modifying Columns:

Changing the data type:
SQL
ALTER TABLE table_name MODIFY column_name new_datatype [column_constraint];

#### Example
SQL
ALTER TABLE customers MODIFY phone_number VARCHAR(15);
Adding or removing constraints (e.g., primary key, unique):
SQL
ALTER TABLE table_name ADD CONSTRAINT constraint_name PRIMARY KEY (column_name);

#### Example
SQL
ALTER TABLE orders ADD CONSTRAINT order_id PRIMARY KEY (order_id);

Deleting Columns:

SQL
ALTER TABLE table_name DROP COLUMN column_name;

#### Example

SQL
ALTER TABLE employees DROP COLUMN address;

#### Closing Summary
+ Common Mistakes/Follow-Ups
Always ensure that changes are well thought out to avoid data inconsistencies or performance issues.

#### Cons
ider the impact on existing queries and applications using the table before making alterations.
Use WITH CHECK OPTION when adding constraints if you want to enforce rules based on current data.

#### Common mistakes
include:

Forgetting to update existing rows if a column's data type is changed, which can lead to data loss or corruption.
Not properly testing changes in a development environment before applying them to production.

**Code:**

```sql
-- Adding a new column
ALTER TABLE employees ADD department_id INTEGER;

-- Modifying an existing column
ALTER TABLE customers MODIFY phone_number VARCHAR(15);

-- Adding a primary key constraint
ALTER TABLE orders ADD CONSTRAINT order_id PRIMARY KEY (order_id);

-- Dropping a column
ALTER TABLE employees DROP COLUMN address;
```

**Flow:**

1. Check
2. Alterations
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Start
10. Adding Columns
11. Add column_name withdatatype
12. Modifying Columns
13. Modify column_name tonew_datatype
14. Deleting Columns
15. Drop column_name
16. End
17. Check data integrity andperformance

