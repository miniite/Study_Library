---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the CHAR and VARCHAR2 data types in Oracle?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Data Types](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Data_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an Oracle database, both CHAR and VARCHAR2 are used to store character strings, but they differ significantly in how they handle storage and performance. The core of this difference lies in their fixed-length and variable-length nature, respectively.

#### Triaging the Question

The question is essentially asking you to explain two fundamental data types in Oracle: CHAR and VARCHAR2. The interviewer might be probing your understanding of basic SQL data types, how they impact storage, and your ability to choose the right type for different scenarios. You should assume a basic understanding of SQL and relational databases.

#### Solution
from First Principles

CHAR Data Type:

Fixed-length: CHAR always allocates a fixed amount of space, regardless of the actual length of the string.

#### Example
If you define a column as CHAR(10), it will always reserve 10 bytes for storage, even if only 5 characters are entered.

VARCHAR2 Data Type:

Variable-length: VARCHAR2 allocates space based on the actual length of the string.

#### Example
If you define a column as VARCHAR2(10), it will store up to 10 bytes, using only the necessary amount.
Full Solution

When deciding between CHAR and VARCHAR2 in Oracle:

Storage Efficiency:

Use VARCHAR2 for more efficient storage when dealing with variable-length strings. It uses less space by only reserving what is needed.

Performance:

VARCHAR2 generally performs better in query execution, especially when you are dealing with large datasets.

#### Cons
istency and Predictability:

Use CHAR when you need consistent storage length and predictable performance. This type is useful for fixed-length data like PINs or ZIP codes where the exact number of characters is always known and constant.

#### Closing Summary

In summary, VARCHAR2 offers more efficient storage and better performance by using only as much space as needed, whereas CHAR reserves a fixed amount of space regardless of content. Choosing between them depends on your specific requirements for storage efficiency and consistency.

#### Common Mistakes/Follow-Ups
Mistake: Assuming both types are interchangeable without considering the implications.

#### Follow-Up
The interviewer might ask about scenarios where CHAR would be preferred over VARCHAR2, such as when you need to ensure a fixed length for all records in a column.

**Code:**

```sql
-- Example of defining CHAR and VARCHAR2 columns
CREATE TABLE example_table (
    char_column CHAR(10),
    varchar2_column VARCHAR2(10)
);
```

