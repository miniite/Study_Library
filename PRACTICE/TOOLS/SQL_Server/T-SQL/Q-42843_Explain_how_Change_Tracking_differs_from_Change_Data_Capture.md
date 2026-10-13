---
type: "practice"
difficulty: "Expert"
---

# Explain how Change Tracking differs from Change Data Capture and when you would pick each for downstream sync.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Temp Tables vs Table Variables](<../../../../TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Change Tracking (CT) and Change Data Capture (CDC) are both mechanisms to track changes in SQL Server, but they serve different purposes and have distinct use cases. In a nutshell, CT is simpler and more efficient for tracking row-level changes within a single table or schema, while CDC provides more comprehensive support for capturing data changes across multiple tables.

To understand the differences and when to choose each approach, let's start from first principles.

**Comparison:**

```
Change Tracking	CDC
Simpler to set up and manage	More complex setup
Smaller storage footprint	Larger storage requirements due to additional tables
Limited to a single table or schema	Supports multiple tables
```

**Code:**

```

-- Example setup for Change Tracking (CT) ALTER TABLE Orders SET (CHANGE_TRACKING = AUTO);

CODE
Copy

```sql
-- Example CDC configuration
EXEC sys.sp_cdc_enable_db;
EXEC sys.sp_cdc_enable_table @source_schema = N'dbo', @source_name = N'Customers';
```

**Example input / output:**

```

Here's a simple example to illustrate:

Input:

Table: Orders
Action 1: Insert row into Orders
Action 2: Update existing row in Orders

Output (CT):

SQL
Copy
SELECT * FROM sys.changes_orders;

Output (CDC):

SQL
Copy
SELECT * FROM cdc.dbo_Orders_CT;
```

**Data:**

```text
The following tables and schemas are typically involved:
- For Change Tracking: `sys.changes`, `sys.tables`
- For CDC: `__$start_changes`, `__$end_changes` in the `cdc` schema
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Change Tracking
6. Is the change tracking for asingle table?
7. Use CT
8. CDC
9. Do you need to capturechanges across multipletables?
10. CDC
11. CT

