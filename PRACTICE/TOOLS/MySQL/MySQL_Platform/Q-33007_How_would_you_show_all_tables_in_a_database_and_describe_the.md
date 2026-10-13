---
type: "practice"
difficulty: "Basic"
---

# How would you show all tables in a database and describe the structure of one table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schemas and Databases](<../../../../TOOLS/MySQL/MySQL_Platform/Schemas_and_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To display all tables in a MySQL database and describe the structure of one specific table, we need to use SQL commands effectively. This task involves understanding how to query information schema views and write basic SQL statements. Let's break it down step-by-step.

#### Triaging the Question

The question is asking for two main tasks:

Listing all tables in a database: This requires us to interact with MySQL’s system databases, specifically the information_schema database.
Describing the structure of one table: This involves querying the metadata stored within the information schema to get details about columns and their properties.

#### Assumptions
You have access to a MySQL database management system.
The user has appropriate permissions to execute these queries.

#### Solution
from First Principles
Listing All Tables in a Database

To list all tables, we can use the TABLES view within the information_schema. This view contains information about all tables present in the specified database.

Identify the target database: If you are working with a specific schema, replace {database_name} with its name.
Query the information_schema.TABLES:
SQL
SELECT TABLE_NAME
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = '{database_name}';

This query will return a list of all table names within the specified database.

Describing the Structure of One Table

To describe the structure of one specific table, we can use another view in information_schema, specifically the COLUMNS view. This provides detailed information about each column including data type and constraints.

Identify the target table: Replace {table_name} with the name of the table you want to inspect.
Query the information_schema.COLUMNS:
SQL
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_DEFAULT, EXTRA
FROM information_schema.COLUMNS
WHERE TABLE_SCHEMA = '{database_name}' AND TABLE_NAME = '{table_name}';

This query will return a detailed structure of the specified table.

#### Closing Summary
and Common Mistakes/Follow-Ups

To recap:

Listing all tables: Use information_schema.TABLES.
Describing one table's structure: Use information_schema.COLUMNS.

#### Common mistakes
include not selecting from the correct views or making typos in SQL queries, especially when dealing with schema and table names.

Final Thoughts

Always ensure you have the necessary permissions to query system tables. Additionally, pay attention to performance impacts of such queries on large databases as they can be resource-intensive due to their nature of accessing metadata.

**Code:**

```sql
-- Listing all tables in a database
SELECT TABLE_NAME
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'your_database_name';

-- Describing the structure of one table
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_DEFAULT, EXTRA
FROM information_schema.COLUMNS
WHERE TABLE_SCHEMA = 'your_database_name' AND TABLE_NAME = 'your_table_name';
```

