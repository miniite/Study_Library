---
type: "practice"
difficulty: "Intermediate"
---

# What is a PL/SQL collection and what are the three collection types?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Functions in SQL vs PL/SQL](<../../../../TOOLS/Oracle/PL-SQL/Functions_in_SQL_vs_PLSQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PL/SQL, collections are data structures that can store multiple elements of the same type. These collections are particularly useful when you need to handle large sets of data or perform operations on a group of similar items. There are three primary collection types in PL/SQL: tables, varrays, and nested tables.

Let’s start with a brief overview:

Tables can store an indefinite number of elements, similar to a database table.
Varrays (Variable-length Arrays) have a fixed maximum size but allow dynamic resizing during the execution of a procedure or function.
Nested Tables are dynamic in nature and do not have a predefined upper limit on their size.

Now let's delve into each collection type:

#### Triaging the Question

The question is asking about PL/SQL collections, specifically focusing on the three main types. The interviewer is likely probing your understanding of data structures in PL/SQL, how they differ from each other, and when to use one over another. Assumptions include familiarity with basic PL/SQL constructs and an understanding that these collection types are part of Oracle's PL/SQL environment.

#### First-Principles Solution
1. PL/SQL Tables
A TABLE is a collection type defined by the TABLE OF keyword.
It can store any number of elements (like a database table), and each element must be of the same type.

#### Example
SQL
DECLARE
  TYPE number_table_type IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  my_numbers number_table_type;
BEGIN
  my_numbers(1) := 5; -- Assigning values
  my_numbers(2) := 10;
END;
2. PL/SQL Varrays
A VARRAY (Variable-length Array) is a collection with a fixed maximum size.
Elements can be added dynamically, but the total number of elements cannot exceed the declared limit.

#### Example
SQL
DECLARE
  TYPE number_varray_type IS VARRAY(10) OF NUMBER;
  my_numbers number_varray_type := number_varray_type(); -- Initialize with no elements
BEGIN
  my_numbers.EXTEND; -- Increase size by 1 element
  my_numbers(1) := 5;
  my_numbers(2) := 10;
END;
3. PL/SQL Nested Tables
A NESTED TABLE is a dynamic collection with no predefined upper limit.
Elements can be added or removed during execution, making it highly flexible.

#### Example
SQL
DECLARE
  TYPE number_table_type IS TABLE OF NUMBER;
  my_numbers number_table_type := number_table_type(); -- Initialize as an empty table
BEGIN
  my_numbers.EXTEND; -- Increase size by 1 element
  my_numbers(1) := 5;
  my_numbers.EXTEND;
  my_numbers(2) := 10;
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the three PL/SQL collection types—tables, varrays, and nested tables—are designed to handle varying amounts of data dynamically within a program. Understanding their differences is crucial for efficient data manipulation.

#### Common mistakes

Confusing VARRAY with TABLE or NESTED TABLE.
Not using dynamic sizing properly in each type (e.g., incorrectly setting the maximum size for VARRAYs).

For follow-ups, you might be asked about:

When to use one collection over another.
How these collections interact with PL/SQL cursors and SQL statements.
Performance implications of different collection types.

