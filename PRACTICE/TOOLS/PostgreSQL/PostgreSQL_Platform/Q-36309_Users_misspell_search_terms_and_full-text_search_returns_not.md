---
type: "practice"
difficulty: "Deep"
---

# Users misspell search terms and full-text search returns nothing; how would you combine tsvector search with pg_trgm similarity for typo tolerance, and where would you draw the line and adopt a real search engine?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Full-text Search in Postgres](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address user misspellings in PostgreSQL full-text search (FTS), we can combine tsvector with pg_trgm similarity to provide typo tolerance. The key idea is that while tsvector excels at fuzzy matching based on word proximity and dictionary rules, it may not catch all typos. pg_trgm, on the other hand, measures substring similarities, which can help identify misspelled words even if they are close to correct ones. By integrating both methods, we create a robust search mechanism that improves user experience.

The question is asking how to combine these two technologies for typo tolerance and where to draw the line between using pg_trgm alone versus implementing a full-fledged search engine like Elasticsearch or Solr. The interviewer is probing your understanding of performance trade-offs, scalability considerations, and real-world implementation nuances.

#### Triaging the Question
Understand the Core Problem: Users frequently misspell search terms, leading to no results from FTS.
Identify the Technologies: tsvector for full-text indexing, pg_trgm for similarity searches.
Performance and Scalability: Assessing when a more advanced solution like Elasticsearch is necessary.

#### First-Principles Solution
Step 1: Configure tsvector

First, ensure that your columns are indexed with a tsvector. For example:

SQL
CREATE INDEX idx_posts_title ON posts USING GIN(to_tsvector('simple', title));
Step 2: Implement Fuzzy Search Using pg_trgm

Next, use the similarity function from pg_trgm to find similar words. This can be done by comparing a user's input with known terms in your database.

SQL
SELECT * FROM posts WHERE (title::text SIMILAR TO '%user_input%');
Step 3: Combine Both Approaches

Combine the two methods using a SQL query that first checks for exact matches and then fuzzy matches:

SQL
-- Exact match check
WITH exact_results AS (
    SELECT id, similarity(title, :user_input) as sim_score FROM posts WHERE to_tsvector('simple', title) @@ plainto_tsquery(:user_input)
),
fuzzy_results AS (
    SELECT * FROM posts WHERE (title::text SIMILAR TO '%' || :user_input || '%')
)
SELECT * FROM exact_results
UNION ALL
SELECT * FROM fuzzy_results EXCEPT SELECT id FROM exact_results;
Step 4: Performance Considerations and Scalability
Indexing: Ensure that both tsvector and pg_trgm indexes are optimized.
Query Optimization: Use appropriate query hints and analyze execution plans.
Drawing the Line Between pg_trgm and a Search Engine

The decision on when to use pg_trgm alone versus adopting a real search engine depends on:

Performance Requirements: If your dataset is small, pg_trgm might suffice.
Scalability: As your dataset grows, FTS can become slower; this is where Elasticsearch shines.
Feature Richness: Full-text indexing features like stemming and stop words removal in Elasticsearch may be necessary.

#### Closing Summary

Combining tsvector with pg_trgm provides a robust solution for typo-tolerant searches in PostgreSQL. However, as your application scales or requires more advanced search capabilities, implementing a real search engine becomes advisable. The choice depends on the trade-offs between performance and feature richness.

#### Common Mistakes/Follow-Ups
Over-relying on pg_trgm: FTS might still miss some typos.
Ignoring Performance: Not tuning indexes or query optimization can lead to poor performance.
Not Considering Scalability: A small-scale solution may not scale well for large datasets.

**Comparison:**

| id  | title          |
|-----|----------------|
| 1   | The quick brown fox |
| 2   | A lazy dog      |

**Code:**

```sql
-- Example of tsvector and pg_trgm usage
CREATE INDEX idx_posts_title ON posts USING GIN(to_tsvector('simple', title));

WITH exact_results AS (
    SELECT id, similarity(title, 'correct_search_term') as sim_score FROM posts WHERE to_tsvector('simple', title) @@ plainto_tsquery('correct_search_term')
),
fuzzy_results AS (
    SELECT * FROM posts WHERE (title::text SIMILAR TO '%' || 'incorrect_search_term' || '%')
)
SELECT * FROM exact_results
UNION ALL
SELECT * FROM fuzzy_results EXCEPT SELECT id FROM exact_results;
```

**Example input / output:**

```text
-- Input: Incorrect search term 'thek quick brown fock'
WITH exact_results AS (
    SELECT id, similarity(title, 'thek quick brown fock') as sim_score FROM posts WHERE to_tsvector('simple', title) @@ plainto_tsquery('thek quick brown fock')
),
fuzzy_results AS (
    SELECT * FROM posts WHERE (title::text SIMILAR TO '%' || 'thek quick brown fock' || '%')
)
SELECT * FROM exact_results
UNION ALL
SELECT * FROM fuzzy_results EXCEPT SELECT id FROM exact_results;

-- Output: []
```

**Data:**

```json
{
  "posts": [
    { "id": 1, "title": "The quick brown fox" },
    { "id": 2, "title": "A lazy dog" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User inputs search term
4. Is typo-tolerant required?
5. Combine tsvector and pg_trgm
6. Check exact matches with FTS
7. Check fuzzy matches withpg_trgm
8. Return combined results
9. Use full-fledged search enginelike Elasticsearch

