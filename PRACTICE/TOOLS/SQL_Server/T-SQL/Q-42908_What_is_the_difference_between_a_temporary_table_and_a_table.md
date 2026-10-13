---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a temporary table and a table variable in T-SQL?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Temp Tables vs Table Variables](<../../../../TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Temporary tables and table variables are both used to store intermediate data in T-SQL, but they differ significantly in their behavior and use cases. A quick summary is that temporary tables are more robust, flexible, and can be accessed by other sessions, whereas table variables offer faster performance but have limitations on size and functionality.

The question probes the understanding of internal storage and lifecycle management of these objects in T-SQL. We need to consider when to use each based on specific requirements such as data size, concurrency needs, and transactional context.

First, let's break down the core concepts:

Temporary Tables: These are tables that exist for the duration of a session or until explicitly dropped.
Table Variables: These are in-memory structures that persist only within a single batch or stored procedure execution.

Now, we'll explore these in detail from first principles:

Temporary Tables

Scope and Lifecycle:

Local temporary tables (#tablename): Visible to the current session only.
Global temporary tables (##tablename): Visible across all sessions that have created them.

Storage:

Typically stored on disk, allowing for a larger data size limit (up to 2GB per table).

Concurrency and Transactions:

Can be used in transactions and support complex operations like joins and updates.
Table Variables

Scope and Lifecycle:

Declared within a batch or stored procedure.

Storage:

Stored in memory, limited to a smaller size (approximately 1MB per table variable).

Concurrency and Transactions:

Not visible outside the current scope of their declaration.
Cannot participate in transactions; changes are committed immediately.

#### Example Scenario
s

Let's consider some typical use cases for each:

Using a Temporary Table
SQL
CREATE TABLE #SalesTemp (
    ID INT,
    ProductName NVARCHAR(100),
    QuantitySold INT
);

INSERT INTO #SalesTemp (ID, ProductName, QuantitySold)
VALUES
(1, 'Laptop', 25),
(2, 'Phone', 45),
(3, 'Tablet', 30);

SELECT * FROM #SalesTemp;

Here, the temporary table #SalesTemp can be used across multiple queries and stored procedures within a session.

Using a Table Variable
SQL
DECLARE @SalesData TABLE (
    ID INT,
    ProductName NVARCHAR(100),
    QuantitySold INT
);

INSERT INTO @SalesData (ID, ProductName, QuantitySold)
VALUES
(1, 'Laptop', 25),
(2, 'Phone', 45),
(3, 'Tablet', 30);

SELECT * FROM @SalesData;

In this example, the table variable @SalesData is limited to the batch or stored procedure where it is declared and cannot be accessed by other parts of the code.

#### Closing Summary

In summary, temporary tables are ideal for storing larger datasets that need to be shared across multiple operations within a session. They offer transactional support and can handle more complex queries. Table variables are better suited for smaller in-memory datasets that do not require sharing or complex transactions but provide faster execution times.

#### Common Mistakes/Follow-Ups
Misusing Temporary Tables: Using temporary tables when table variables would be sufficient, leading to unnecessary overhead.
Ignoring Concurrency Issues: Not considering the visibility and concurrency implications of global temporary tables.
Failing to Clean Up: Forgetting to drop temporary tables at the end of a session can lead to memory leaks.

