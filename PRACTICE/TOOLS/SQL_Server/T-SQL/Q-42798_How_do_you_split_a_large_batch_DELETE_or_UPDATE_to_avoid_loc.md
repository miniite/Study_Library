---
type: "practice"
difficulty: "Deep"
---

# How do you split a large batch DELETE or UPDATE to avoid lock escalation and keep the transaction log from exploding?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ MERGE Pitfalls and Error Handling](<../../../../TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with large batches of data in T-SQL, it's essential to manage locks and the transaction log effectively. This is particularly important when performing operations like DELETE or UPDATE on a massive dataset. By splitting the operation into smaller batches, you can mitigate lock escalation and reduce the load on the transaction log.

#### Triaging the Question

The core of this question revolves around optimizing T-SQL operations to avoid excessive locking and transaction logging. Specifically:

Lock Escalation: We need to prevent large locks from being escalated to table-level locks.
Transaction Log Management: Minimize the size of the transaction log by reducing the number of log entries.

#### Assumptions
The operation involves a very large dataset that cannot be handled in one go due to performance or resource constraints.
The database is under heavy load, and you need to ensure minimal impact on other transactions.
You have access to the necessary permissions and tools to execute T-SQL commands.

#### First-Principles Solution

To achieve this, we can break down the operation into smaller batches using a combination of ROW_NUMBER, PARTITION BY, or TOP with WITH (TABLOCKX). This approach ensures that each batch is processed in its own transaction scope, reducing lock escalation and minimizing the log footprint.

Step-by-Step Solution
Partitioning Data: Use a common column to partition the data into smaller chunks.
Batch Processing: Execute individual batches within separate transactions.
Lock Management: Ensure each batch is processed with minimal locking by using appropriate transaction isolation levels and hints.

#### Example
Code
SQL
-- Assume we are working on a table named LargeTable for DELETE operations

DECLARE @batchSize INT = 1000; -- Adjust based on your dataset size and performance requirements
DECLARE @counter INT = 1;

WHILE EXISTS (SELECT TOP 1 1 FROM LargeTable)
BEGIN
    WITH BatchedData AS (
        SELECT ROW_NUMBER() OVER (ORDER BY SomeColumn) AS RowNum,
               *
        FROM LargeTable
    )
    DELETE FROM BatchedData
    WHERE RowNum BETWEEN (@counter - 1) * @batchSize + 1 AND @counter * @batchSize;

    SET @counter = @counter + 1;
END;

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By breaking down the operation into smaller batches, you can manage locks more effectively and keep the transaction log from growing excessively. This approach ensures that each batch is processed in a separate transaction with minimal locking.

#### Common Mistakes
Failing to partition the data properly, leading to unnecessary full table scans.
Not adjusting @batchSize based on the performance characteristics of your dataset.
Ignoring proper isolation levels and hints, which can lead to excessive locking.

#### Follow-Ups
Performance Monitoring: Use SET SHOWPLAN_ALL ON or SQL Profiler to monitor the execution plan and ensure that each batch is indeed processed efficiently.
Logging Strategy: Consider implementing a logging strategy that captures the progress of your operation for auditing and troubleshooting purposes.
Index Management: Ensure that appropriate indexes are in place to support efficient partitioning and deletion operations.

**Comparison:**

```
table "LargeTable" as lt {
    rowNum INT
    someColumn VARCHAR
    dataColumn1 INT
    dataColumn2 VARCHAR
}
```

**Code:**

```sql
-- Example T-SQL code block
```

**Example input / output:**

| Batch | StartRowNum | EndRowNum |
|-------|-------------|-----------|
| 1     | 1           | 1000      |
| 2     | 1001        | 2000      |
| ...   | ...         | ...       |

**Data:**

```json
{
  "LargeTable": {
    "columns": ["SomeColumn", "DataColumn1", "DataColumn2"],
    "rows": [
      { "RowNum": 1, "SomeColumn": "A", "DataColumn1": 100, "DataColumn2": "Value1" },
      { "RowNum": 2, "SomeColumn": "B", "DataColumn1": 150, "DataColumn2": "Value2" },
      { "RowNum": 3, "SomeColumn": "C", "DataColumn1": 200, "DataColumn2": "Value3" }
    ]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Is Batch Exists?
5. End
6. Assign RowNum to BatchSize
7. Process Batch with DELETEStatement
8. Log Operation Completion

