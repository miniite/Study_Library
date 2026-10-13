---
type: "practice"
difficulty: "Intermediate"
---

# A search feature uses LIKE 'term%' and is fast, but 'contains' search with LIKE '%term%' is slow — how do you explain the index difference and plan a fulltext migration?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Full-text Search](<../../../../TOOLS/MySQL/MySQL_Platform/Full-text_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with two types of SQL queries on a MySQL database: one using LIKE 'term%' which performs well due to an index, and another using LIKE '%term%' which is slow because it does not utilize the index effectively. The key difference lies in how indexes are used by these patterns.

TL;DR

The query LIKE 'term%' uses a prefix search, which can leverage an index efficiently. In contrast, LIKE '%term%' performs a full-text search, which cannot use indexes and is slower because it requires scanning the entire table.

#### Triaging the Question
Understanding the Issue: The question revolves around performance differences between two types of SQL queries: prefix (LIKE 'term%') and full-text (LIKE '%term%') searches.
Key Probes: The interviewer likely wants to assess your understanding of:
Indexes and how they work in MySQL.
Full-text indexing and its limitations.

#### Assumptions
We assume that the database has an appropriate index on the column being searched for prefix queries, but no full-text index is present.

#### First-Principles Solution
Prefix Search (LIKE 'term%'):
This type of query can leverage a B-tree index because it starts with a specific character and looks forward.
MySQL can efficiently scan the index to find rows that match the prefix term.
Full-Text Search (LIKE '%term%'):
This pattern does not benefit from an index, as it requires scanning the entire table or using a full-text index.
Full-text searches are more complex and generally involve tokenization, stemming, and other processes that do not align with simple LIKE queries.
Steps to Explain
Index Utilization:
For LIKE 'term%', MySQL can use an index on the column containing 'term'. The database can quickly narrow down the search space using the prefix.
Full-Text Indexing:
To improve performance for full-text searches, we should consider adding a FULLTEXT index on the relevant columns.

#### Implementation
Plan:
Identify columns suitable for full-text indexing.
Add appropriate FULLTEXT indexes to those columns.
Adjust queries to use MATCH AGAINST instead of LIKE '%term%'.

#### Closing Summary

By leveraging the differences between prefix and full-text searches, we can significantly improve query performance. Prefix searches like LIKE 'term%' benefit from standard B-tree indexes, while full-text searches require specialized indexing. Migrating to a full-text index will help in optimizing queries that use wildcards at the end.

#### Common Mistakes/Follow-Ups
Misunderstanding Index Usage: Not recognizing why LIKE 'term%' uses an index and how it differs from LIKE '%term%'.
Incorrect Index Type: Using regular indexes where full-text indexing is more appropriate.
Performance Degradation: Failing to realize that the current solution with a simple prefix search is already efficient compared to full-text searches.

Would you like me to walk through an example of creating a full-text index and running a query?

**Comparison:**

| Column     | Type           | Nullable | Default      | Extra       |
|------------|----------------|----------|--------------|-------------|
| id         | int(10)        | NO       | auto_increment | primary_key |
| title      | varchar(255)   | YES      | NULL         |            |
| content    | text           | YES      | NULL         |             |

**Code:**

```sql
-- Example table creation
CREATE TABLE articles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT
);

-- Adding a FULLTEXT index on the column that needs full-text search
ALTER TABLE articles ADD FULLTEXT(title, content);

-- Query example using MATCH AGAINST
SELECT * FROM articles WHERE MATCH (title, content) AGAINST ('term' IN NATURAL LANGUAGE MODE);
```

**Example input / output:**

```text
Input: SELECT * FROM articles WHERE MATCH (title, content) AGAINST ('term' IN NATURAL LANGUAGE MODE);

Output:
+----+-------------------------+----------------------------------+
| id | title                   | content                          |
+----+-------------------------+----------------------------------+
| 1  | How to Build a Database | A guide to database development...|
+----+-------------------------+----------------------------------+
```

**Data:**

```json
{
  "table": {
    "articles": [
      { "id": 1, "title": "How to Build a Database", "content": "A guide to database development..." },
      { "id": 2, "title": "Optimizing Search Queries", "content": "Tips for improving search performance..." }
    ]
  }
}
```

**Flow:**

1. Query 'LIKE 'term%''
2. Uses index on prefix
3. Query 'LIKE '%term%''
4. Scans full table
5. Add FULLTEXT index
6. Improves performance for LIKE'%term%'

