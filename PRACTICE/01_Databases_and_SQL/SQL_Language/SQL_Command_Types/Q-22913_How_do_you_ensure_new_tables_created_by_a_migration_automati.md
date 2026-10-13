---
type: "practice"
difficulty: "Intermediate"
---

# How do you ensure new tables created by a migration automatically inherit the correct GRANTs without manual steps each time?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that newly created tables in a database automatically inherit the correct GRANTS without manual steps, we can leverage Database Definition Language (DDL) and Database Management System (DBMS) features like TRIGGERs or INSTEAD OF triggers to enforce this inheritance. This solution ensures that new tables are created with the necessary permissions attached seamlessly.

#### Triage

The question asks how to automatically apply GRANTS to newly created tables, which involves understanding both DDL for creating tables and DCL (Data Control Language) for managing permissions. The goal is to automate the process of setting up these GRANTs without manual intervention each time a new table is created through migrations or other means.

#### First-Principles Solution
Identify the Migration Process: Understand how new tables are being added to the database, whether it's via scripts, ETL processes, or migration tools.
DDL and GRANTs: Ensure that your DDL scripts include a step for applying GRANTS when creating new tables.
Triggers or INSTEAD OF Triggers: Implement triggers that automatically apply necessary GRANTS to newly created tables.
Step-by-Step Solution

Understand the Migration Process:

If you use SQL migration tools like Liquibase, Flyway, or custom scripts, integrate a step in these processes to enforce GRANTs.

DDL and GRANTs:

SQL
CREATE TABLE new_table (
    id INT,
    name VARCHAR(100),
    PRIMARY KEY (id)
);

-- Apply GRANTS immediately after creating the table
GRANT SELECT, INSERT ON new_table TO 'user_role';

Triggers for Automation:

Use a BEFORE or AFTER CREATE TABLE trigger to automatically apply necessary GRANTS.
SQL
DELIMITER //

CREATE TRIGGER ensure_grants BEFORE CREATE TABLE ON schema_name
FOR EACH ROW BEGIN
    -- Example: Apply SELECT and INSERT permissions
    EXECUTE IMMEDIATE 'GRANT SELECT, INSERT ON ' || NEW.table_name || ' TO ''user_role''';
END;//

DELIMITER ;

Testing the Trigger:

Create a new table to test if the trigger works as expected.
SQL
CREATE TABLE test_table (
    id INT,
    name VARCHAR(100)
);

-- Check permissions
SHOW GRANTS FOR 'user_role'@'localhost';

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing a combination of DDL and triggers, we can ensure that new tables inherit the correct GRANTS automatically. Common mistakes include forgetting to integrate this logic into migration scripts or not thoroughly testing the triggers in different scenarios.

Common Follow-Up Questions:

How would you handle cascading GRANTS if multiple tables are involved?
What security considerations should be taken into account when applying such permissions automatically?

**Code:**

```sql
-- Example DDL and GRANTs
CREATE TABLE new_table (
    id INT,
    name VARCHAR(100),
    PRIMARY KEY (id)
);

GRANT SELECT, INSERT ON new_table TO 'user_role';

-- Trigger to apply GRANTS on table creation
DELIMITER //

CREATE TRIGGER ensure_grants BEFORE CREATE TABLE ON schema_name
FOR EACH ROW BEGIN
    EXECUTE IMMEDIATE 'GRANT SELECT, INSERT ON ' || NEW.table_name || ' TO ''user_role''';
END;//

DELIMITER ;
```

**Example input / output:**

```text
-- Create a new table and check GRANTS
CREATE TABLE test_table (
    id INT,
    name VARCHAR(100)
);

SHOW GRANTS FOR 'user_role'@'localhost';
```

