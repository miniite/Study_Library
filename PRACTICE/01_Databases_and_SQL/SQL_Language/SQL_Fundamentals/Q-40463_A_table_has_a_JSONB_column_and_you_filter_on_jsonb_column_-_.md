---
type: "practice"
difficulty: "Deep"
---

# A table has a JSONB column and you filter on jsonb_column ->> 'status' constantly. Compare a GIN index, an expression B-tree index on the extracted key, and promoting status to a real column. Which for high-selectivity equality vs. containment queries?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Values and TOAST](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the question of comparing a GIN index, an expression B-tree index on the extracted key, and promoting status to a real column in a JSONB column with high-selectivity equality vs. containment queries, we need to consider several factors: query patterns, data distribution, performance metrics, and maintenance overhead.

The core of this problem is understanding how different types of indexes behave under various conditions. Let's break it down step by step from first principles.

**Code:**

```sql
-- Example table with JSONB column
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    jsonb_column JSONB
);
```

**Example input / output:**

```

Equality Query Example:

SQL
Copy
-- GIN Index
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM events WHERE jsonb_column ->> 'status' = 'active';

-- Expression B-tree Index on extracted key
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM events WHERE (jsonb_column->>'status')::text = 'active';

-- Promoting status to a real column
ALTER TABLE events ADD COLUMN status text;
UPDATE events SET status = jsonb_column->>'status';
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM events WHERE status = 'active';

Containment Query Example:

SQL
Copy
-- GIN Index
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM events WHERE jsonb_column @> '{"status": "active"}';

-- Expression B-tree Index is not directly useful for containment queries as it's built on equality.

-- Promoting status to a real column
ALTER TABLE events ADD COLUMN status text;
UPDATE events SET status = jsonb_column->>'status';
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM events WHERE status IN ('active', 'inactive');
```

**Data:**

```json
[
    {"id": 1, "jsonb_column": {"status": "active", "name": "Event A"}},
    {"id": 2, "jsonb_column": {"status": "inactive", "name": "Event B"}},
    {"id": 3, "jsonb_column": {"status": "completed", "name": "Event C"}}
]
```

