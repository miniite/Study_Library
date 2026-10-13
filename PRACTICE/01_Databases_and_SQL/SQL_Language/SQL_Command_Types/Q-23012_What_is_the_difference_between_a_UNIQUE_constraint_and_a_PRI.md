---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a UNIQUE constraint and a PRIMARY KEY defined in DDL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, understanding the differences between UNIQUE constraints and PRIMARY KEY definitions in Data Definition Language (DDL) is fundamental to ensuring data integrity and efficient schema design. Let's break down these concepts step by step.

#### Triaging the Question

The question asks us to distinguish between a UNIQUE constraint and a PRIMARY KEY. Both are essential for defining constraints on columns or sets of columns, but they serve different purposes. The primary difference lies in their behavior regarding nullability, uniqueness enforcement, and how they interact with other SQL operations.

#### First-Principles Solution
UNIQUE Constraint

A UNIQUE constraint ensures that a set of one or more columns contains no duplicate values. It can be applied to any column or combination of columns. Here are some key points:

Nullability: A column defined with UNIQUE can contain nulls, and multiple nulls can coexist in the same table.
Enforcement: Ensures uniqueness but does not guarantee a single entry for every unique value. This means that duplicates of non-null values will be allowed.
PRIMARY KEY

A PRIMARY KEY, on the other hand, is used to uniquely identify each record in a database table. It has specific characteristics:

Nullability: A column defined as a PRIMARY KEY cannot contain nulls.
Uniqueness: Ensures that all values are unique and no duplicates can exist (including non-null).
Enforcement: Acts as both a uniqueness constraint and an index, which means it provides faster lookup times for records.

#### Example
Illustration

Let's illustrate the differences with some sample data:

SQL
CREATE TABLE Users (
    id INT,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100),
    PRIMARY KEY (id)
);

In this example:

username is defined as a UNIQUE constraint, allowing multiple rows to have the same username but ensuring no nulls.
email does not have any specific constraint and can contain duplicate or null values.
id serves as the PRIMARY KEY, ensuring it cannot be null and must uniquely identify each row.

#### Closing Summary

In summary, a UNIQUE constraint is used to ensure that certain columns do not contain duplicate values, while allowing for nulls. In contrast, a PRIMARY KEY enforces both uniqueness and non-nullability on the specified column(s). Understanding these differences helps in designing efficient and effective database schemas.

#### Common Mistakes
and Follow-Ups
Misunderstanding Null Values: Often, developers mistakenly believe that a UNIQUE constraint allows nulls only if multiple rows have them. However, it’s crucial to remember that a single null value is not permitted.
Ignoring Indexes: While both UNIQUE and PRIMARY KEY can create indexes, the PRIMARY KEY automatically creates an index for faster lookups, whereas a UNIQUE constraint might need explicit indexing depending on usage.

#### Example
Code
SQL
-- Creating a table with UNIQUE and PRIMARY KEY constraints
CREATE TABLE Users (
    id INT,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100),
    PRIMARY KEY (id)
);

