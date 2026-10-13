---
type: "practice"
difficulty: "Deep"
---

# Why can passing an NVARCHAR parameter against a VARCHAR column defeat an index, and how do collation and data-type precedence drive it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Filtered Indexes and Plan Cache](<../../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL Server, passing an NVARCHAR parameter to a query that filters on a VARCHAR column can defeat an index due to differences in collation handling and data type precedence. This occurs because NVARCHAR is a Unicode string type, while VARCHAR is not. In this scenario, the engine may have to perform a full table scan or use a different approach that doesn't utilize the index, leading to suboptimal performance.

#### Triage
The Question

The question delves into the nuances of how SQL Server handles data types and collation settings during query execution. Specifically, it examines why an NVARCHAR parameter can prevent the use of an existing index on a VARCHAR column. We need to understand how these factors influence the optimizer's decision-making process.

#### First-Principles Solution
1. Collation Implications

Collation is a set of rules for sorting and comparing character data, which includes case sensitivity and accent sensitivity. When you pass an NVARCHAR parameter to a query filtering on a VARCHAR column, SQL Server must first convert the NVARCHAR value to match the collation of the VARCHAR column.

#### Example
If a table has a VARCHAR(50) column with the default collation SQL_Latin1_General_CP1_CI_AS, and you pass an NVARCHAR parameter, SQL Server will convert this parameter to VARCHAR. However, if the collation of the NVARCHAR value doesn't match the collation of the VARCHAR column, a non-matching conversion occurs.
2. Data Type Precedence

SQL Server follows certain rules when deciding which data type is most appropriate for operations:

Implicit Conversion: SQL Server will implicitly convert the smaller data type to the larger one if necessary.
Explicit Conversion: You can also explicitly cast or convert types using functions like CAST and CONVERT.

When an NVARCHAR parameter is passed, it needs to be converted to match the type of the column. This conversion causes the query optimizer to consider other execution plans, often leading to a full table scan.

3. Index Utilization

An index on a VARCHAR column can be used effectively when the data types and collations are compatible:

Exact Match: If both the parameter and the indexed column have the same type and collation, the index can be used efficiently.
Type Incompatibility: When there is a mismatch due to an NVARCHAR parameter, SQL Server might choose a different execution plan that does not use the index.

#### Closing Summary

Passing an NVARCHAR parameter against a VARCHAR column can lead to suboptimal query performance because of the conversion overhead and potential collation mismatches. The query optimizer will consider these factors when determining whether to use an existing index or opt for a full table scan, thereby defeating the benefits of the index.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring data type and collation settings can lead to performance issues.

#### Follow-Up
Question: How can you ensure that NVARCHAR parameters are handled correctly with VARCHAR columns in queries?

Mistake: Failing to consider the impact of implicit conversions on query performance.

#### Follow-Up
Question: Can you provide an example where explicit conversion might be beneficial?

**Comparison:**

```
Column Name	Data Type
Id	INT
Description	VARCHAR
```

**Code:**

```sql
-- Example of a VARCHAR column and NVARCHAR parameter
CREATE TABLE ExampleTable (
    Id INT PRIMARY KEY,
    Description NVARCHAR(100) -- Note: should be VARCHAR if the data is not Unicode
)

-- Query with NVARCHAR parameter
DECLARE @Param NVARCHAR(50) = N'example';

SELECT *
FROM ExampleTable
WHERE Description = @Param;
```

**Example input / output:**

```text
Input:
@Param = N'example'

Output:
No rows returned or full table scan performed.
```

**Data:**

```sql
CREATE TABLE ExampleTable (
    Id INT PRIMARY KEY,
    Description VARCHAR(100)
)

INSERT INTO ExampleTable (Id, Description)
VALUES (1, 'example');
```

**Flow:**

1. Yes
2. No
3. Query with NVARCHARparameter
4. Is the column typecompatible?
5. Use index
6. Full table scan or differentexecution plan

