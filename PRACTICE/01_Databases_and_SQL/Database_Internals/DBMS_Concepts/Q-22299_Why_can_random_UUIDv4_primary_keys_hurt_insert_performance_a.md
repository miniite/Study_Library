---
type: "practice"
difficulty: "Expert"
---

# Why can random UUIDv4 primary keys hurt insert performance, and how does UUIDv7 or ULID help?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1 · [↑ Composite vs Surrogate Keys](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using a database management system (DBMS), it's crucial to understand that the choice of primary key impacts the efficiency of data operations, especially during insertions. In the case of UUIDv4, while they offer excellent uniqueness and portability across systems, their random distribution can lead to performance issues in certain scenarios. This response will explore why UUIDv4 can hurt insert performance and how UUIDv7 (or ULID) addresses these concerns.

Firstly, let's understand what a primary key is: it uniquely identifies each row in a database table. The choice of the primary key affects not only data integrity but also query performance, particularly during write operations like inserts. In traditional databases, integer auto-increment keys are often used due to their sequential nature, which helps maintain a sorted index and optimize insert performance.

UUIDv4, on the other hand, generates 128-bit identifiers that are random by design. This randomness means they do not follow any particular pattern or order, leading to non-uniform distribution when inserted into the database. For large tables, this can result in poor insertion performance because the DBMS may need to perform more expensive disk I/O operations and potentially fragment the index.

UUIDv7 (or ULID) addresses these issues by combining a timestamp with random data. This hybrid approach results in keys that are both unique and have a better distribution pattern than UUIDv4, which helps maintain better performance during inserts.

To illustrate this, consider inserting 10 million records into a table. With UUIDv4, the index might become unbalanced due to its randomness, leading to slower insert operations as more disk I/O is required for each new entry. In contrast, with UUIDv7 or ULID, the keys will have a roughly linear distribution over time, which can significantly improve insert performance and maintain an efficient B-tree index.

