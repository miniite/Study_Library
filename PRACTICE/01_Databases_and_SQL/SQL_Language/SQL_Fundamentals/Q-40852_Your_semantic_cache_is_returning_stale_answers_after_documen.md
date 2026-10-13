---
type: "practice"
difficulty: "Deep"
---

# Your semantic cache is returning stale answers after documents changed. How would you invalidate cache rows in SQL when the underlying source chunk's version bumps?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of stale data in a semantic cache caused by changes to the underlying document versions, we need to ensure that our cache is invalidated or updated whenever a relevant document's version is bumped. This involves identifying the dependencies between the cached content and the source chunks, and then implementing an SQL-based mechanism to manage this relationship effectively.

Triaging the Problem

The core question here is how to invalidate rows in a semantic cache when the underlying document versions change. The key aspects being probed are:

Dependencies: How the cache relies on specific documents.
Versioning: Managing version bumps and their impact on cached data.
SQL Mechanisms: Using SQL queries or triggers to update or invalidate rows based on these dependencies.

#### Assumptions
We have a semantic cache that stores derived information from documents.
Documents are represented in the database with a version field.
Each cached row references one or more document chunks via foreign keys.
The source of truth for document versions is stored in a separate table, say Documents, which contains columns like id, version, and any other relevant metadata.

#### First Principles Solution
Identify Dependencies: Determine which documents are referenced by each cache entry. This could be done using foreign keys or some form of association.
Track Version Changes: Monitor the version changes in the Documents table. When a version is bumped, we need to invalidate or update corresponding cache rows.
SQL Mechanisms:
Use SQL triggers to automatically handle invalidation when a document's version changes.
Implement a periodic cleanup process using SQL queries to ensure stale entries are removed.
Step-by-Step Solution

Foreign Key Associations: Ensure that each cache entry has a foreign key linking it to the associated documents.

Document Table Schema:

SQL
CREATE TABLE Documents (
    id INT PRIMARY KEY,
    version INT NOT NULL,
    ...
);

Cache Table Schema:

SQL
CREATE TABLE CacheEntries (
    id INT PRIMARY KEY,
    document_id INT,
    chunk_version INT,  -- Reference to the specific chunk version used in the cache entry
    ...
);

Trigger for Version Changes: Create a trigger that invalidates or updates cache entries when a document's version changes.

SQL
CREATE TRIGGER invalidate_cache ON Documents FOR UPDATE AS
BEGIN
    -- Invalidate all cache entries associated with the updated document
    DELETE FROM CacheEntries WHERE document_id IN (SELECT id FROM inserted);
END;

Cleanup Process: Implement a periodic job to clean up outdated cache entries.

SQL
CREATE PROCEDURE CleanCache @timestamp DATETIME
AS
BEGIN
    DELETE FROM CacheEntries WHERE chunk_version < (
        SELECT version
        FROM Documents
        WHERE id = CacheEntries.document_id AND last_modified <= @timestamp
    );
END;

Trigger for Document Insertion: Ensure that new documents are properly associated with cache entries.

SQL
CREATE TRIGGER associate_cache ON Documents FOR INSERT AS
BEGIN
    -- Insert into CacheEntries if the document is relevant to existing cache logic
    INSERT INTO CacheEntries (document_id, chunk_version)
    SELECT id, version FROM inserted WHERE id IN (SELECT document_id FROM CacheLogic);
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Failing to properly associate documents with cache entries, not using triggers for dynamic updates, or missing periodic cleanup.

#### Follow-Ups
Discussing the performance implications of frequent invalidation and how to optimize it, such as batching deletions. Additionally, exploring caching strategies like read-through caching or distributed caches that might provide more granular control over stale data.

