---
type: "practice"
difficulty: "Basic"
---

# How do you add a new column to an existing table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add a new column to an existing table, we use Data Definition Language (DDL) commands in SQL, specifically the ALTER TABLE statement combined with the ADD COLUMN clause. This operation is essential when you need to modify your database schema without affecting the data within the table.

#### Triaging the Question

The question is asking about adding a new column to an existing table using SQL. The interviewer may be probing your understanding of basic SQL commands and their syntax, as well as how these operations impact the database structure. We will assume that we are working with a relational database system like MySQL or PostgreSQL.

Step-by-Step Solution
Identify the Table: First, identify the name of the existing table where you want to add the new column.
Define the Column: Determine the data type and any additional constraints for the new column (e.g., NOT NULL, DEFAULT value).
Write the SQL Command: Use the ALTER TABLE statement followed by the ADD COLUMN clause with appropriate syntax.

#### Example

Suppose we have a table named employees in our database:

SQL
-- Example DDL for creating an existing employees table
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    department_id INT
);

Now, if we want to add a new column salary of type DECIMAL(10, 2) with a default value of 0.0, the SQL command would be:

SQL
ALTER TABLE employees ADD COLUMN salary DECIMAL(10, 2) DEFAULT 0.0;

#### Closing Summary
and Common Mistakes

Adding a new column to an existing table using DDL is straightforward but critical for maintaining database integrity. The key steps are identifying the table, defining the column properly, and ensuring the correct syntax in SQL.

#### Common Mistakes

Forgetting to define data type constraints.
Not handling potential issues like NOT NULL constraints that might require setting default values or providing initial values.
Failing to consider the impact on indexes if the new column is added as part of an index definition (though such a step would be covered in more advanced questions).

**Code:**

```sql
ALTER TABLE employees ADD COLUMN salary DECIMAL(10, 2) DEFAULT 0.0;
```

