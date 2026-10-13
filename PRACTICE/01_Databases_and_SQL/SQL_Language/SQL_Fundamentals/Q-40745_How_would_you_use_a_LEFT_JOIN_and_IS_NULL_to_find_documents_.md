---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a LEFT JOIN and IS NULL to find documents that have zero embedded chunks in your knowledge base?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/RAG_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we'll leverage SQL's LEFT JOIN and IS NULL clauses to identify documents that lack any associated chunks in our knowledge base. The key idea is to join the document table with the chunk table, ensuring all documents are included even if they don't have corresponding entries in the chunk table.

#### Triaging the Question

The question asks us to find documents that have zero embedded chunks. In SQL terms, this means identifying rows in the documents table where there are no matching records in the chunks table for a given document ID (or some identifier). We'll achieve this using a LEFT JOIN, which returns all records from the left table (documents) and the matched records from the right table (chunks). Any mismatched records in the right table will result in NULL values.

#### Assumptions
The documents table has a primary key, such as document_id.
The chunks table also has a foreign key to documents, say chunk_document_id.
We want to identify documents that have no associated chunks.

#### First-Principles Solution

To implement this in SQL, we'll perform the following steps:

Join: Perform a LEFT JOIN between the documents and chunks tables.
Filter: Use an IS NULL condition on the foreign key of the chunks table to filter out documents that have no associated chunks.

Here is the step-by-step solution in SQL:

SQL
SELECT d.*
FROM documents d
LEFT JOIN chunks c ON d.document_id = c.chunk_document_id
WHERE c.chunk_document_id IS NULL;

#### Closing Summary
and Common Mistakes/Follow-Ups

The query will return all document records that do not have any associated chunk entries. This is a fundamental SQL operation used to handle cases where we need to check for the absence of related data.

#### Common Mistakes

Incorrect Join Condition: Ensure you are joining on the correct keys.
Misunderstanding LEFT JOIN: Confusing it with an INNER JOIN, which would not include documents without associated chunks.

#### Follow-Ups

What if we want to find documents that have at least one chunk?
How can this be optimized for large datasets?

