---
type: "practice"
difficulty: "Deep"
---

# Can a primary key column be a foreign key at the same time? Give an example.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Foreign Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Foreign_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A primary key column cannot directly be used as a foreign key in the same table. However, it can functionally serve a similar purpose if you have multiple tables with identical structures and relationships. Let's explore this concept from first principles.

#### Triaging the Question

The question asks whether a single column can simultaneously hold roles of both a primary key (PK) and a foreign key (FK). In SQL databases, these are distinct concepts:

Primary Key: Identifies unique rows in a table.
Foreign Key: References a column or columns in another table.

#### Assumptions
The database is relational (SQL-based).
We are dealing with standard normalization practices.

#### First-Principles Solution
Primary Key vs Foreign Key

A primary key must be unique and non-null, while a foreign key can be null if there's no corresponding value in the referenced table. Typically, these roles are assigned to different columns for clarity and integrity enforcement.

However, consider a scenario where you have multiple tables with identical schemas, or when you want to enforce relationships within a single table without duplicating data. In such cases, the primary key of one table can act as a foreign key in another, though this is not a common practice due to potential issues like cascading updates and referential integrity.

#### Example

Let's assume we have two tables: Users and Profiles. Both could share the same structure:

SQL
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    email VARCHAR(255)
);

CREATE TABLE Profiles (
    profile_id INT PRIMARY KEY,
    user_id INT,  -- This can be a foreign key to `Users`
    bio TEXT
);

In this case:

user_id in the Profiles table references user_id in the Users table.
user_id cannot be both a primary key and a foreign key within the same table, as that would violate normalization principles.

#### Closing Summary

While it is technically possible to reuse a column for both PK and FK roles across different tables, doing so within the same table leads to potential issues. Typically, you should avoid such practices unless there's a specific need, like managing relationships between entities in a highly denormalized structure or performance optimization scenarios.

#### Common Mistakes/Follow-Ups
Misunderstanding Roles: Mixing up the roles of PK and FK within the same table.
Normalization Violations: Ignoring normalization principles can lead to data redundancy and inconsistency.
Referential Integrity Issues: Using a column as both PK and FK might cause cascading updates or other referential integrity problems.

Would you like an example with more complexity, or are there any follow-up questions on this topic?

**Code:**

```sql
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    email VARCHAR(255)
);

CREATE TABLE Profiles (
    profile_id INT PRIMARY KEY,
    user_id INT,  -- Foreign key to Users
    bio TEXT
);
```

