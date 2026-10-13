---
type: "practice"
difficulty: "Intermediate"
---

# Your text search on a description column uses to_tsvector on every query and it's slow; how would you add a generated tsvector column with a GIN index and keep it in sync automatically?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Full-text Search in Postgres](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To improve performance of text searches on a PostgreSQL table, we need to reduce the overhead of generating to_tsvector every time a query is run. Adding a generated tsvector column and creating a GIN index on it will allow us to cache the vectorized data, making full-text searches much faster. Let’s break down how to implement this.

TL;DR
Add a computed tsvector column to your table.
Create a GIN index on the newly added tsvector column.
Ensure that the tsvector column is automatically updated when the relevant text data changes.
Modify your queries to use the new indexed column.

#### Triaging the Question

The core of this question revolves around optimizing full-text search performance in PostgreSQL by leveraging generated columns and indexes. The interviewer is probing your understanding of:

Full-text indexing mechanisms in PostgreSQL (e.g., to_tsvector, GIN index).
Schema design principles.
Index maintenance and automatic updates.

#### First-Principles Solution
1. Add a Computed Column for tsvector

We’ll use a generated column to store the tsvector representation of the description text. This ensures that we always have an up-to-date vectorized version without querying to_tsvector.

SQL
ALTER TABLE my_table
ADD COLUMN tsvector_column TSVECTOR GENERATED ALWAYS AS (to_tsvector('english', description)) STORED;
2. Create a GIN Index on the Generated Column

A GIN index is well-suited for full-text search as it supports fast, partial scans and does not store duplicate values.

SQL
CREATE INDEX idx_tsvector ON my_table USING GIN (tsvector_column);
3. Ensure Automatic Updates

To ensure that the tsvector column updates automatically when the description changes, we can use triggers or AFTER UPDATE logic in our application code to recompute and update the column.

SQL
CREATE OR REPLACE FUNCTION update_tsvector_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.tsvector_column := to_tsvector('english', NEW.description);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Assuming a trigger is not needed in this simple case, as PostgreSQL updates the column on insert/update.
4. Modify Queries to Use the Indexed Column

Now, we can modify our queries to use the tsvector_column instead of to_tsvector.

SQL
SELECT * FROM my_table WHERE tsvector_column @@ to_tsquery('interesting & query');

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, adding a generated tsvector column with a GIN index will significantly improve the performance of your full-text search queries. However, ensure that you test thoroughly after making these changes:

Performance Impact: Measure the impact on both read and write performance.
Index Size: GIN indexes can be larger than B-tree indexes due to their nature.
Maintenance Overhead: Regularly analyze and vacuum the index to maintain optimal performance.

#### Common mistakes
include:

Failing to create a trigger or application logic to update the tsvector column when needed.
Not testing the impact on query performance before implementing these changes.
Ignoring potential issues with large text fields, as to_tsvector can be computationally expensive for very long texts.

#### Follow-up
questions might include:

How would you handle different languages or custom stop words?
What are other indexing options available in PostgreSQL for full-text search?

