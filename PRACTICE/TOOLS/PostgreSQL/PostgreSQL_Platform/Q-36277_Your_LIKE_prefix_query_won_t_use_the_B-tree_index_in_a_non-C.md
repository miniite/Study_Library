---
type: "practice"
difficulty: "Deep"
---

# Your LIKE 'prefix%' query won't use the B-tree index in a non-C locale; explain why and how a text_pattern_ops operator class index fixes prefix matching under a locale-aware collation.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Full-text Search in Postgres](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with LIKE 'prefix%' queries in PostgreSQL, understanding the nuances of indexing can be crucial. In this scenario, we are faced with an interesting issue where a B-tree index fails to optimize such queries when using non-C locales. Let's delve into why this happens and how to fix it.

#### Triaging the Question

The question is primarily about understanding why LIKE 'prefix%' queries might not benefit from a B-tree index in certain configurations, specifically when the database locale is not set to C (which uses simple byte-based comparisons). Additionally, we need to explain the role of the text_pattern_ops operator class and how it can be used with locale-aware collations to achieve efficient prefix matching.

Key Assumptions
Non-C Locale: The locale in use affects string comparison by introducing culture-specific rules.
B-tree Indexes: These indexes are optimized for exact matches, range scans, but not for LIKE 'prefix%' type queries without wildcards at the end.
text_pattern_ops Operator Class: This operator class is designed to handle pattern matching in a way that respects collation rules.

#### First-Principles Solution
Why B-tree Indexes Fail with Non-C Locales

B-tree Index Design: A B-tree index stores data by the byte value of each key, which means it can efficiently find exact matches or ranges but not pattern matching unless the query uses an operator that supports such patterns (like = for equality).

Locale-Specific Comparisons: In non-C locales, strings are compared using locale-specific rules. For example, a German 'ß' might be considered equivalent to 'ss' according to certain collation settings.

LIKE Query Behavior: The LIKE 'prefix%' query relies on the pattern-matching behavior of the database engine. When the locale is not C, the collation rules can change how strings are compared, making it difficult for a B-tree index to provide useful results without additional context.

Using text_pattern_ops Operator Class

text_pattern_ops: This operator class extends the functionality of the LIKE and ILIKE operators by providing pattern matching behavior that respects collation rules.

Collation Support: By using an appropriate collation, we can ensure that string comparisons are consistent with the locale's specific rules.

Index Creation: We create a GiST (Generalized Search Tree) index on the column(s) of interest using text_pattern_ops as the operator class. This allows PostgreSQL to use advanced search techniques tailored for pattern matching.

#### Example
Solution
SQL
-- Set up the locale and collation
SET lc_collate = 'de_DE.UTF-8';
SET lc_ctype = 'de_DE.UTF-8';

CREATE TABLE test_table (
    id SERIAL PRIMARY KEY,
    value TEXT
);

-- Insert some data
INSERT INTO test_table (value) VALUES ('foo'), ('foöo'), ('fóo');

-- Create an index using text_pattern_ops
CREATE INDEX idx_test ON test_table USING gist (value text_pattern_ops);

-- Query with LIKE 'prefix%'
EXPLAIN ANALYZE SELECT * FROM test_table WHERE value LIKE 'fo%';

#### Closing Summary

By understanding the limitations of B-tree indexes in non-C locales and leveraging the text_pattern_ops operator class, we can achieve efficient prefix matching that respects locale-specific rules. The key is to ensure that the index creation matches the query patterns and the collation settings.

#### Common Mistakes/Follow-Ups
Failing to Set Collation: Not setting the correct collation can lead to unexpected behavior.
Incorrect Index Type: Using a B-tree index for LIKE 'prefix%' queries without wildcards at the end may not provide significant benefits, and other types of indexes like GiST might be more appropriate.
Ignoring Locale Settings: Forcing all operations into the C locale might simplify some issues but can ignore important cultural differences in string comparison.

**Comparison:**

| Column | Data Type |
|--------|----------|
| id     | SERIAL   |
| value  | TEXT     |

**Code:**

```sql
-- Set up the locale and collation
SET lc_collate = 'de_DE.UTF-8';
SET lc_ctype = 'de_DE.UTF-8';

CREATE TABLE test_table (
    id SERIAL PRIMARY KEY,
    value TEXT
);

INSERT INTO test_table (value) VALUES ('foo'), ('foöo'), ('fóo');

-- Create an index using text_pattern_ops
CREATE INDEX idx_test ON test_table USING gist (value text_pattern_ops);
```

**Example input / output:**

```text
EXPLAIN ANALYZE SELECT * FROM test_table WHERE value LIKE 'fo%';
-- Expected output: Index Scan using idx_test on test_table
-- Time: [1ms] (actual time)
```

**Data:**

```json
[
  { "id": 1, "value": "foo" },
  { "id": 2, "value": "foöo" },
  { "id": 3, "value": "fóo" }
]
```

**Flow:**

1. Set Locale and Collation
2. Set lc_collate = 'de_DE.UTF-8';SET lc_ctype = 'de_DE.UTF-8'
3. Create Table
4. Insert Data
5. Create Index Using gist withtext_pattern_ops
6. Explain Analyze Query

