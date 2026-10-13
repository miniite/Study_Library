---
type: "practice"
difficulty: "Expert"
---

# What are In-Memory OLTP tables and natively compiled procedures, and what workloads justify them despite their restrictions?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Temp Tables vs Table Variables](<../../../../TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In-Memory OLTP (Online Transaction Processing) tables and natively compiled T-SQL procedures in SQL Server offer significant performance benefits by leveraging memory-optimized storage and just-in-time compilation, respectively. These features are highly beneficial for transactional workloads that require high throughput and low latency but come with certain restrictions such as limited support for complex queries and specific data types.

Firstly, let's understand what In-Memory OLTP tables are and when they justify their use:

What are In-Memory OLTP Tables? They store data in memory rather than on disk, which significantly reduces the I/O operations required to access or modify data. This makes them ideal for high-frequency read/write operations.
Justification: Workloads that frequently update or insert rows and require minimal query complexity (e.g., financial transactions, reservations systems) benefit most from In-Memory OLTP tables.

Now, let’s discuss natively compiled procedures:

What are Natively Compiled Procedures? These are T-SQL stored procedures that are compiled as native machine code at deployment time. This compilation process can lead to significant performance improvements.
Justification: They are best for procedural logic where the performance gain from inlining and optimizing the code outweighs the limitations, such as complex queries or large datasets.

In-Memory OLTP tables and natively compiled procedures have specific restrictions:

Restrictions on In-Memory OLTP Tables:

Limited data types (no LOB columns, sparse columns, or XML data).
No support for certain T-SQL features like common table expressions (CTEs) and table-valued functions.
Restrictions on indexing.

Restrictions on Natively Compiled Procedures:

Limited procedural logic allowed; no cursors, while loops, and dynamic SQL.
Cannot use some built-in scalar functions or methods that are not supported in native compilation.

To summarize the key points:

In-Memory OLTP Tables should be used when high transactional throughput and low latency are crucial.
Natively Compiled Procedures provide significant performance benefits for complex procedural logic but require careful design to avoid exceeding their limitations.

**Comparison:**

```
InMemOrder	OrderID	ProductID	Quantity	Price
#InMemOrders	1	1001	5	$9.99
```

**Code:**

```

-- Example of creating an In-Memory OLTP table CREATE TABLE #InMemOrders ( OrderID INT IDENTITY PRIMARY KEY NONCLUSTERED, ProductID INT NOT NULL, Quantity INT NOT NULL, Price MONEY NOT NULL ) WITH (MEMORY_OPTIMIZED = ON);

CODE
Copy

```sql
-- Example of a natively compiled procedure
ALTER PROCEDURE #OrderSummary @StartDate DATE, @EndDate DATE AS
NATIVE_COMPILATION
BEGIN ATOMIC
    SELECT SUM(Quantity * Price) as TotalSales
    FROM InMemOrders
    WHERE OrderDate BETWEEN @StartDate AND @EndDate;
END;
```

**Example input / output:**

```text
Input:
EXEC #OrderSummary '2023-01-01', '2023-01-31'

Output:
TotalSales = $4,985.00
```

**Data:**

```json
{
  "InMemOrder": {
    "OrderID": 1,
    "ProductID": 1001,
    "Quantity": 5,
    "Price": 9.99
  }
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Initial Query
6. Is the query complex?
7. Use In-Memory OLTP Table
8. Can it be natively compiled?
9. Compile as Natively CompiledProcedure
10. Fallback to Traditional TableStorage

