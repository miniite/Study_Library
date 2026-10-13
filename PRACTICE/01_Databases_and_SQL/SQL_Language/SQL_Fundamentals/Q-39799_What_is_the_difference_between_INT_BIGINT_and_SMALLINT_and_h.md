---
type: "practice"
difficulty: "Basic"
---

# What is the difference between INT, BIGINT, and SMALLINT, and how do you choose the right one for an id column?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Storing Money](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL databases, INT, BIGINT, and SMALLINT are data types used to store integer values. They differ in terms of their range and precision, which affects how you choose them for an ID column based on your specific requirements.

#### Triage
What the Question is Asking

The question aims to understand the differences between these three data types and when it's appropriate to use each one for an ID column. The interviewer is probing your knowledge of SQL data types, your understanding of precision and range limitations, and how to make informed decisions based on those constraints.

Step-by-Step Solution
INT Data Type

INT (or INTEGER) is a 4-byte signed integer. This means it can store values from -2,147,483,648 to 2,147,483,647.

SQL
-- Example of creating a table with INT type ID column
CREATE TABLE example_table (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);
BIGINT Data Type

BIGINT is an 8-byte signed integer. It can store values from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807.

SQL
-- Example of creating a table with BIGINT type ID column
CREATE TABLE example_table (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100)
);
SMALLINT Data Type

SMALLINT is a 2-byte signed integer. This means it can store values from -32,768 to 32,767.

SQL
-- Example of creating a table with SMALLINT type ID column
CREATE TABLE example_table (
    id SMALLINT PRIMARY KEY,
    name VARCHAR(100)
);
Choosing the Right Data Type
Key Considerations
Range Requirements: If you need to store IDs that could potentially grow beyond 32,767 (the maximum value of SMALLINT), then INT or BIGINT should be used. For most applications, INT is a good default choice as it provides a wide enough range for typical use cases.
Memory Usage: Smaller data types like SMALLINT take up less memory and can improve performance in large-scale databases where millions of rows are expected. However, the difference in storage space between SMALLINT, INT, and BIGINT is relatively small (2 bytes for SMALLINT, 4 bytes for INT, and 8 bytes for BIGINT).
Performance: For very large databases or when performance is critical, using a smaller data type can help reduce the overall storage footprint and potentially speed up queries.
Best Practices
Use INT by Default: Unless you have specific requirements that necessitate a larger or smaller range, INT is generally sufficient.

#### Cons
ider Future Growth: If your application might support a large number of entities (e.g., users, products), consider using BIGINT to ensure the ID column will not overflow in the future.

#### Closing Summary

In summary, choosing between INT, BIGINT, and SMALLINT for an ID column depends on the range requirements, memory considerations, and performance needs of your application. By understanding the differences and their implications, you can make an informed decision that best fits your database schema.

#### Common Mistakes/Follow-Ups
Overestimating Range Needs: Choosing a data type with an excessively large range can lead to unnecessary storage overhead.
Underestimating Future Growth: Assuming current requirements will remain constant might lead to ID overflow issues in the future.
Ignoring Performance Considerations: Failing to consider how smaller or larger ID columns affect overall performance, especially in very large databases.

**Code:**

```sql
CREATE TABLE example_table (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE example_table (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE example_table (
    id SMALLINT PRIMARY KEY,
    name VARCHAR(100)
);
```

