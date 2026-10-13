---
type: "practice"
difficulty: "Basic"
---

# How do triggers show up in a database schema, and how would a new developer discover them?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML vs DDL Triggers](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/DML_vs_DDL_Triggers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Triggers are special database objects that automatically execute when specific events occur on a table. A new developer can find triggers by querying the system catalog or using administrative tools provided by the database management system (DBMS).

#### Triaging the Question

The question is asking how a new developer would identify and understand triggers in a database schema. The interviewer might be probing your knowledge of:

How triggers are stored and accessed within a DBMS.
The steps involved in discovering triggers programmatically or through administrative tools.

#### Assumptions
We are using a relational database system like MySQL, PostgreSQL, or SQL Server.
The developer has basic access to the database management environment (e.g., command line or GUI).

#### Solution
from First Principles

Triggers in a database schema are essentially pieces of code that get attached to specific events on a table. To find them, you can follow these steps:

Understand Trigger Concepts:

A trigger is typically defined within the context of a single table.
Triggers can be associated with DML statements such as INSERT, UPDATE, or DELETE.
They execute automatically when one of these events occurs on the table.

Locating Triggers in System Catalogs:

Most DBMSes have their own system catalogs that store metadata about triggers.
In PostgreSQL, you can use \dt to list tables and then query pg_trigger.
In MySQL, you might explore the information_schema.triggers table.

Using Administrative Tools:

Many DBMSes provide GUI tools for database administration where one can view triggers directly.
For example, in pgAdmin (for PostgreSQL), you can browse the schema and see all objects, including triggers.

Writing SQL Queries to Discover Triggers:

SQL
-- Example query to list triggers in a PostgreSQL database
SELECT trigger_name, event_manipulation, table_name
FROM information_schema.triggers;

#### Closing Summary
+ Common Mistakes/Follow-Ups
A common mistake is assuming that triggers are always visible without querying the system catalog. Triggers are internal objects and not directly accessible via standard SQL queries on tables.
Always check the documentation of your specific DBMS to understand how to query for triggers.

#### Follow-Up
Questions
How do you handle conflicts when multiple triggers are defined on a single table?
Can triggers be used in stored procedures? If so, what's an example use case?

**Code:**

```sql
-- Example SQL query for PostgreSQL
SELECT trigger_name, event_manipulation, table_name
FROM information_schema.triggers;
```

