---
type: "practice"
difficulty: "Basic"
---

# How do you make a column store only unique email addresses when creating the table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Unique Constraints for Dedup](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a specific column in a database table stores only unique email addresses, we need to create a constraint or index on that column during the table creation process. This is a fundamental aspect of SQL (Structured Query Language) and involves using either a UNIQUE constraint or an appropriate index.

#### Triage
The question asks how to enforce uniqueness on an email address column when creating a table. We need to use SQL commands to define this constraint properly.

Key Assumptions:

You are familiar with basic SQL syntax.
You have access to the database management system (DBMS) and can run SQL scripts.

Step-by-step Solution:

Understanding Uniqueness: A unique constraint ensures that each value in a specified column is distinct across all rows of the table, meaning no two rows will have the same email address.

SQL Syntax for Creating a Table with a Unique Constraint:

Use the CREATE TABLE statement to define your table.
Specify the column where you want to enforce uniqueness using the UNIQUE keyword.

Here is an example of how to create such a table:

SQL
CREATE TABLE Users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE
);

In this example:

id is set as the primary key, which ensures each row has a unique identifier.
The email column is marked with the UNIQUE keyword to ensure no duplicate values.
Creating the Table: Once you run the above SQL script in your DBMS (e.g., MySQL, PostgreSQL), it will create the table and enforce the uniqueness constraint on the email column.

#### Closing Summary
By using the UNIQUE keyword during the creation of a table or altering an existing table, we can ensure that only unique values are stored in specified columns. This is essential for maintaining data integrity and preventing duplication issues.

#### Common Mistakes/Follow-ups

Not Understanding Indexes: Sometimes developers mistakenly think adding a primary key is enough to enforce uniqueness when they need a UNIQUE constraint.
Misplacement of Constraints: Placing constraints on the wrong column can lead to errors or unintended behavior in the database schema.

#### Performance Considerations
While unique constraints are necessary, they may impact performance during insertions and updates. Indexes should be carefully planned.

**Code:**

```sql
CREATE TABLE Users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE
);
```

