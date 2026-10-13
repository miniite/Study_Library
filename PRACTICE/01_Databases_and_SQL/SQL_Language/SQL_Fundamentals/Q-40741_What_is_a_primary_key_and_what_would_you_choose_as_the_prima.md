---
type: "practice"
difficulty: "Basic"
---

# What is a primary key, and what would you choose as the primary key for a table of embedded chunks?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Foreign Keys and Referential Integrity](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A primary key in SQL uniquely identifies each record in a database table. For an embedded chunk table, we need to ensure that every chunk's unique identifier is captured while considering factors like efficiency and uniqueness.

When designing the primary key for a table of embedded chunks, you must understand what constitutes a "chunk." Typically, chunks refer to segments of data, such as text or binary content, stored in a database. The goal is to create an identifier that uniquely represents each chunk without redundancy.

#### Triage
The question asks us to define the primary key for an embedded chunks table. This involves understanding the nature of the chunks and ensuring that every entry can be identified uniquely within the database. The primary key should also consider performance, storage efficiency, and ease of query operations.

#### Assumptions
We assume each chunk has a unique identifier or content, and we aim to avoid generating new identifiers if they are not strictly necessary.

#### First-principles solution

Understanding the Chunks Table Structure:

The table stores information about each embedded chunk, such as its ID, content, type (e.g., text, binary), metadata (e.g., creation time, user ID).

Defining the Primary Key:

In SQL, a primary key should be unique and non-null.
For an embedded chunks table, we can choose the chunk_id as the primary key. This field is typically generated during the upload process or assigned by the system.

Alternative Considerations:

If chunk_id does not exist, we could consider using a combination of other fields like (content_hash, creation_time) to ensure uniqueness.
However, this approach may lead to additional storage overhead and complexity in ensuring the hash is unique across all chunks.

#### Example
Schema:

SQL
CREATE TABLE embedded_chunks (
    chunk_id BIGINT NOT NULL PRIMARY KEY,
    content BLOB,
    type VARCHAR(50),
    metadata JSONB,
    creation_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

#### Closing summary
and common mistakes/follow-ups:

#### Common Mistakes

Choosing a non-unique field as the primary key, leading to data integrity issues.
Overcomplicating the primary key with unnecessary fields, which can degrade performance.

#### Follow-Ups

How would you handle updates or deletions of chunks?
What if two chunks have identical content but different metadata?

**Code:**

```sql
CREATE TABLE embedded_chunks (
    chunk_id BIGINT NOT NULL PRIMARY KEY,
    content BLOB,
    type VARCHAR(50),
    metadata JSONB,
    creation_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

