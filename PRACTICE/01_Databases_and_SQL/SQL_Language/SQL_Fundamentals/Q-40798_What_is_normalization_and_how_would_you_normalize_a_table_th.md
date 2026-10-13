---
type: "practice"
difficulty: "Basic"
---

# What is normalization, and how would you normalize a table that repeats the full document title on every chunk row?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/RAG_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Normalization in SQL is the process of organizing data to minimize redundancy and dependency. It ensures data integrity by splitting complex tables into smaller, more structured ones. This prevents duplicate data storage, which can lead to inconsistencies.

The scenario you've described involves a table that stores document chunks where each row repeats the full document title. To normalize this, we need to separate the document title from the chunk content and establish a relationship between them. Here's how you would approach it step-by-step:

Identify the Entities: The entities here are the Document and its Chunks.
Create Separate Tables: Create a Documents table to store document titles, IDs, and other metadata. Create a Chunks table to store individual chunks along with their references back to the parent documents.
Reference Integrity: Use foreign keys in the Chunks table to reference the Documents table.

Let's break down these steps:

#### Triage
The question is asking how to normalize an existing schema that repeats document titles across all rows, leading to redundancy and potential inconsistencies.
We need to create a more normalized structure by separating out the repeated data into its own table while maintaining relationships with the original data.
Step-by-Step Solution from First Principles

Identify Primary Entities:

Documents: Stores unique document information.
Chunks: Stores content chunks and references their parent documents.

Define Tables:

SQL
CREATE TABLE Documents (
    DocumentID INT PRIMARY KEY,
    Title VARCHAR(255),
    -- Other metadata fields can be added here
);

CREATE TABLE Chunks (
    ChunkID INT PRIMARY KEY,
    Content TEXT,
    DocumentID INT,
    FOREIGN KEY (DocumentID) REFERENCES Documents(DocumentID)
);

Insert Sample Data:

SQL
INSERT INTO Documents (DocumentID, Title) VALUES
(1, 'Introduction to SQL Normalization'),
(2, 'Advanced SQL Techniques');

INSERT INTO Chunks (ChunkID, Content, DocumentID) VALUES
(1, 'This is the first chunk.', 1),
(2, 'Second paragraph of the first document.', 1),
(3, 'Third paragraph of the second document.', 2);

Query to Retrieve Data:

SQL
SELECT D.Title, C.Content
FROM Documents D
JOIN Chunks C ON D.DocumentID = C.DocumentID;

#### Closing Summary
and Common Mistakes/Follow-Ups
The normalized approach ensures that the document title is stored only once in the Documents table. This reduces redundancy and improves data integrity.

#### Common mistakes
include not defining relationships properly (e.g., missing foreign keys or incorrect references).

#### Follow-up
questions could involve performance considerations, storage savings, and whether additional indexes might be needed.

**Code:**

```sql
CREATE TABLE Documents (
    DocumentID INT PRIMARY KEY,
    Title VARCHAR(255)
);

CREATE TABLE Chunks (
    ChunkID INT PRIMARY KEY,
    Content TEXT,
    DocumentID INT,
    FOREIGN KEY (DocumentID) REFERENCES Documents(DocumentID)
);
```

**Data:**

```sql
INSERT INTO Documents (DocumentID, Title) VALUES
(1, 'Introduction to SQL Normalization'),
(2, 'Advanced SQL Techniques');

INSERT INTO Chunks (ChunkID, Content, DocumentID) VALUES
(1, 'This is the first chunk.', 1),
(2, 'Second paragraph of the first document.', 1),
(3, 'Third paragraph of the second document.', 2);
```

