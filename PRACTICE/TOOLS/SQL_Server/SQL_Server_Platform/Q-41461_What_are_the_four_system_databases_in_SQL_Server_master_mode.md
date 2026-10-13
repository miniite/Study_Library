---
type: "practice"
difficulty: "Basic"
---

# What are the four system databases in SQL Server (master, model, msdb, tempdb) and what is each one used for?

**Difficulty:** Basic · **Levels:** Fresher · [↑ MERGE and Upserts](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/MERGE_and_Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a basic SQL Server setup, there are four critical system databases that come pre-installed with every instance: master, model, msdb, and tempdb. These databases serve fundamental roles in the database management system's operation.

Firstly, let’s break down what each of these databases is used for:

master Database: This database contains crucial information about the SQL Server instance itself. It stores metadata such as server-level objects like logins, servers, and linked servers.

model Database: The model database acts as a template or blueprint for new user databases created on that instance. Any changes made to the structure of the model database get applied when a new user database is created.

msdb Database: This database primarily stores information related to SQL Server Agent jobs, backup and restore operations, and other maintenance tasks. It's essential for scheduling and managing tasks within an environment.

tempdb Database: The tempdb database serves as a shared workspace for all user sessions on the SQL Server instance. Temporary tables, table variables, and temporary result sets are stored here.

Now let’s delve into more details about each:

Master Database

The master database is critical because it holds essential metadata required by the SQL Server engine to function correctly. For example:

Server-Level Objects: Logins, server roles, linked servers.
System Catalog Information: Indexes for system catalogs and dynamic management views (DMVs).

Without a healthy master database, many administrative tasks will fail, as it contains vital information needed to manage the SQL Server instance.

Model Database

The model database is used primarily during the creation of new user databases. Any changes made to its schema are applied when creating a new user database. This means that if you want certain default settings or objects in every database created on your server, they can be defined here and will automatically get included.

For example:

Default Schema: Defining a default schema for new users.
Table Structures: Creating templates of tables that should exist by default in all user databases.
Msdb Database

The msdb database is heavily used by the SQL Server Agent. It stores information about jobs, alerts, and schedules. Key features include:

Jobs and Schedules: Information about scheduled tasks and their execution history.
Backups and Restores: Details of backup operations, including logs, files, and related metadata.

This database is crucial for any environment where automated maintenance or administrative tasks are necessary.

Tempdb Database

The tempdb database is a shared resource among all user sessions. It’s very important because:

Temporary Tables and Table Variables: Stores data that needs to be temporary and session-specific.
Table Joins and Operations: Often used for intermediate results in complex queries where temporary storage is needed.

Since it's shared, it can become a bottleneck if not managed properly. Frequent cleanup of unused objects can help maintain performance.

#### Closing Summary
+ Common Mistakes

Understanding the roles of these system databases is fundamental to managing SQL Server effectively. A common mistake beginners might make is neglecting tempdb management, leading to performance issues due to excessive usage or lack of optimization. Another frequent oversight is not recognizing the importance of the model database when setting up new user databases.

**Comparison:**

| Database | Description                       |
|----------|-----------------------------------|
| master   | Contains server-level metadata    |
| model    | Used as a template for new DBs    |
| msdb     | Stores info related to jobs/backups|
| tempdb   | Shared temporary database         |

**Code:**

```sql
-- Example SQL queries for reference

-- View contents of master database
SELECT * FROM sys.databases WHERE name = 'master';

-- View contents of model database
SELECT * FROM sys.databases WHERE name = 'model';

-- View contents of msdb database
SELECT * FROM sys.databases WHERE name = 'msdb';

-- View contents of tempdb database (tempdb is a system database and cannot be directly queried)
```

**Example input / output:**

```text
Input:
SELECT * FROM sys.databases WHERE name IN ('master', 'model', 'msdb', 'tempdb');

Output:
| database_name |
|---------------|
| master        |
| model         |
| msdb          |
| tempdb        |
```

**Data:**

```json
{
  "master": {
    "description": "Contains server-level metadata"
  },
  "model": {
    "description": "Used as a template for new user databases"
  },
  "msdb": {
    "description": "Stores information related to SQL Server Agent jobs and backups"
  },
  "tempdb": {
    "description": "Shared temporary database used by all sessions"
  }
}
```

**Flow:**

1. Master Database
2. Server-Level Objects
3. System Catalog Information
4. Model Database
5. Default Schema
6. Table Structures
7. MSSQL
8. Msdb Database
9. Backups and Restores
10. T-SQL
11. TempDB Database
12. Temporary Tables and TableVariables

