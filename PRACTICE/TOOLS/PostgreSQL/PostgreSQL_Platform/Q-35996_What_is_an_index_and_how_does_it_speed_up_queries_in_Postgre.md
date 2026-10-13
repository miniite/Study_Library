---
type: "practice"
difficulty: "Basic"
---

# What is an index and how does it speed up queries in PostgreSQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Indexing Multi-join Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a nutshell, an index in PostgreSQL is like having a table of contents in a book—it allows you to quickly find information without reading every page, making queries faster. This concept can be understood from first principles by considering how data is stored and retrieved.

#### Triage
The question asks for an explanation of what an index is and how it speeds up queries in PostgreSQL. The interviewer likely wants to ensure that the candidate understands the basic concepts behind indexing and its impact on query performance.

#### Assumptions
We assume you have a basic understanding of how databases store data, particularly in PostgreSQL.

Step-by-Step Solution:

Data Storage: In PostgreSQL, data is stored in tables as rows and columns. When you issue a SQL query, the database needs to search through these rows to find the relevant data.
Indexing Concept: An index is essentially a data structure that improves the speed of data retrieval operations on a table. Think of it like an index at the back of a book, where each entry points directly to the page number.
B-Tree Indexes: PostgreSQL primarily uses B-tree indexes by default. A B-tree is a type of balanced tree data structure where every leaf node is at the same depth (or height). Each index entry contains the value and the corresponding row's location in the table.
Query Execution: When you run a query, the database first checks the index to find the relevant rows. This process is much faster than scanning through all the rows of the table because the index entries are already ordered by key values.
Insert/Update Operations: While indices speed up read operations, they can slow down write operations like inserts and updates due to the need to maintain the index structure.

#### Closing Summary
In PostgreSQL, an index is a powerful tool for accelerating query performance. By organizing data in a B-tree or similar structure, it allows the database engine to quickly locate relevant rows without having to scan through every row in the table.

#### Common Mistakes/Follow-Ups

Ignoring Indexes: Some developers might ignore indexing because they think it uses too much space and slows down writes. However, for frequently queried tables, proper indexing is essential.
Choosing the Wrong Index Type: Sometimes, a different type of index (like a hash or gist index) might be more appropriate depending on the query patterns.
Over-indexing: Adding too many indexes can slow down write operations and increase storage overhead. The key is to carefully analyze the schema and query patterns.

**Code:**

```sql
-- Example of creating an index in PostgreSQL
CREATE INDEX idx_name ON customers(name);
```

