---
type: "practice"
difficulty: "Basic"
---

# What are the main categories of SQL commands (DDL, DML, DCL, TCL) and give one example of each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Reading EXPLAIN](<../../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

SQL commands can be broadly categorized into four types: Data Definition Language (DDL), Data Manipulation Language (DML), Data Control Language (DCL), and Transaction Control Language (TCL). Understanding these categories is fundamental as they guide how you interact with the database. Let's break down each category and provide an example for clarity.

Data Definition Language (DDL): This category includes commands that are used to create, alter, or drop the structure of a database.
Data Manipulation Language (DML): DML commands are used to insert, update, delete, and retrieve data from a table in the database.
Data Control Language (DCL): These commands handle permissions and access control within the database, determining who can do what with the data.
Transaction Control Language (TCL): TCL commands manage transactions, ensuring that operations are committed or rolled back as needed.

Now let's dive into each category with an example:

#### Example
1: DDL

DDL is used to define and alter the structure of a database. An example would be creating a table.

SQL
CREATE TABLE employees (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    hire_date DATE
);

#### Example
2: DML

DML is used to manipulate data in the database. An example would be inserting a new record into a table.

SQL
INSERT INTO employees (id, first_name, last_name, email, hire_date)
VALUES (1, 'John', 'Doe', 'john.doe@example.com', '2023-01-01');

#### Example
3: DCL

DCL is used to control who can access the data. An example would be granting SELECT permission on a table.

SQL
GRANT SELECT ON employees TO 'user1'@'localhost';

#### Example
4: TCL

TCL commands are used to manage transactions, ensuring that database operations are atomic, consistent, isolated, and durable (ACID properties). An example would be starting a transaction and rolling it back if an error occurs.

SQL
START TRANSACTION;
-- Some SQL statements here...
ROLLBACK; -- or COMMIT;

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Understanding the categories of SQL commands is crucial for effective database management. Remember, DDL creates and modifies the structure, DML manipulates data, DCL controls access, and TCL manages transactions. A common mistake is using DML to alter table structures instead of DDL. Additionally, improper use of TCL can lead to inconsistent or unrecoverable states.

**Code:**

```sql
-- Example DDL
CREATE TABLE employees (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    hire_date DATE
);

-- Example DML
INSERT INTO employees (id, first_name, last_name, email, hire_date)
VALUES (1, 'John', 'Doe', 'john.doe@example.com', '2023-01-01');

-- Example DCL
GRANT SELECT ON employees TO 'user1'@'localhost';

-- Example TCL
START TRANSACTION;
-- Some SQL statements here...
ROLLBACK; -- or COMMIT;
```

