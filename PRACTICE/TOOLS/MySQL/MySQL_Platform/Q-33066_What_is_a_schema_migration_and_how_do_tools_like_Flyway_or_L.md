---
type: "practice"
difficulty: "Intermediate"
---

# What is a schema migration and how do tools like Flyway or Liquibase keep environments in sync?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Online Schema Changes](<../../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

Schema migrations are crucial for managing changes to your database schema over time, ensuring that all development, staging, and production environments have consistent database structures. Tools like Flyway or Liquibase help automate this process by providing a version-controlled approach to manage database schema evolution.

#### Triage

The question is essentially asking how schema migrations work in practice and the role of tools such as Flyway or Liquibase in facilitating these processes. The core idea revolves around ensuring that database schemas evolve consistently across different environments without manual errors, which can be complex if done by hand.

#### Assumptions
You have a basic understanding of relational databases (e.g., MySQL).
You are familiar with the concept of version control systems.
Your application is using an RDBMS like MySQL for storing data.

#### First-Principles Solution

What is a Schema Migration?

A schema migration involves making changes to your database schema (e.g., adding tables, columns, or altering existing ones) in a controlled and repeatable manner.
These changes are managed through scripts that can be executed against the database.

Manual vs Automated Migrations:

Without tools like Flyway or Liquibase, manual migrations require developers to write SQL scripts or commands directly on the target environment, which is error-prone and inconsistent across environments.
Tools automate this process by providing a structured way to manage these changes, ensuring consistency.

How Flyway Works:

Version Control: Flyway treats database schema changes as if they were file revisions in a version control system. Each migration is stored in a directory with a unique ID and script name.
Initialization Script: A V1__Initial_Schema.sql script sets up the initial schema, ensuring that all environments start from the same point.
Migration Scripts: Subsequent migrations are stored as separate scripts (e.g., V2__Add_Products_Table.sql). These scripts are executed in order to apply changes.

#### Example
of Flyway Workflow:

Suppose you have a new feature that requires adding a products table.
SQL
-- V2__Add_Products_Table.sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL
);
When Flyway is run, it will first check if the environment is up to date. If not, it will execute V1__Initial_Schema.sql and then V2__Add_Products_Table.sql.

Rollbacks:

Rollback scripts are also provided in case a migration needs to be undone. For example:
SQL
-- V3__Remove_Old_Columns.sql
ALTER TABLE products DROP COLUMN old_column;
Flyway can execute rollback scripts to undo changes if necessary.

Concurrency and Locking:

To handle concurrent access, Flyway uses a locking mechanism that prevents other migrations from running until the current one is complete.
This ensures that only one migration script runs at a time, preventing conflicts in environments where multiple teams might be working simultaneously.

Integration with CI/CD Pipelines:

Tools like Flyway integrate well with CI/CD pipelines, allowing for automated schema migrations during deployments.
By committing changes to the flyway directory alongside application code, you ensure that all environments are kept in sync automatically.

#### Example
of Liquibase Workflow (Similar but Different):

While Flyway treats scripts as version-controlled files, Liquibase uses XML or YAML configuration files to define schema changes.
This approach provides more flexibility and can handle complex transactions more easily.
An example configuration might look like:
XML
<changeSet id="1" author="developer">
    <createTable tableName="products">
        <column name="id" type="int" />
        <column name="name" type="varchar(100)" />
        <column name="price" type="decimal" />
    </createTable>
</changeSet>

#### Closing Summary

Schema migrations are essential for maintaining consistent database schemas across environments. Tools like Flyway or Liquibase automate this process by providing a structured approach to managing schema changes, ensuring consistency and reducing the risk of errors.

#### Common Mistakes/Follow-Ups
Forgetting Rollback Scripts: Always ensure rollback scripts are available if you need to undo migrations.
Not Versioning Properly: Keeping migration IDs properly versioned can help in tracking which environments have applied specific changes.
Concurrency Issues: Ensure that your tool of choice handles concurrency well, especially in large-scale deployments.

#### Example
Input/Output:

Suppose the current schema is as follows:

SQL
-- Initial Schema (V1__Initial_Schema.sql)
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

Adding a new products table using Flyway might look like this:

SQL
-- Migration Script (V2__Add_Products_Table.sql)
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL
);

Running Flyway will apply these changes if the environment is not up to date.

**Comparison:**

```
Initial Schema
V1__Initial_Schema.sql
No
Migration Applied?
V2__Add_Products_Table.sql
Updated Schema
```

**Code:**

```sql
-- V1__Initial_Schema.sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

-- V2__Add_Products_Table.sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL
);
```

**Example input / output:**

```text
Initial Schema:
+----------------+
| users          |
+----+----------+---------+
| id  | name     | email   |
+----+----------+---------+

Migration to Add Products Table:
+---------------+
| products      |
+----+----------+------+
| id  | name     | price |
+----+----------+------+
```

**Data:**

```sql
-- Initial Schema
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

-- Migration to Add Products Table
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL
);
```

**Flow:**

1. No
2. Environment Setup
3. Flyway Initialization Script(V1__Initial_Schema.sql)
4. Is Environment Up to Date?
5. Migrate Changes (e.g.,V2__Add_Products_Table.sql)
6. Rollback Scripts for UndoneChanges

