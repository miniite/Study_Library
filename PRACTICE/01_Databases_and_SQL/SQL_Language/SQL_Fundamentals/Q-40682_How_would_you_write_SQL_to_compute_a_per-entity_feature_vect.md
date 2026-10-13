---
type: "practice"
difficulty: "Deep"
---

# How would you write SQL to compute a per-entity feature vector as a JSON/array column for cheap serving fetch, and what are the trade-offs versus one column per feature?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Values and TOAST](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a per-entity feature vector as a JSON/array column in SQL for efficient serving, you would typically aggregate multiple feature values into a single row for each entity. This approach reduces the number of rows and simplifies retrieval during inference. The key steps involve selecting the relevant features, aggregating them into an array, and storing this in a JSON or array column.

The core challenge is balancing storage efficiency with query flexibility. SQL’s native support for arrays can make aggregation straightforward but may affect performance depending on the database system and how the feature vectors are queried later.

**Comparison:**

```
id	name	feature_vector
1	entity1	[1.0, 2.0, 3.0]
2	entity2	[4.0, 5.0, 6.0]
```

**Code:**

```sql
-- Example schema
CREATE TABLE entities (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    feature_vector JSONB  -- PostgreSQL JSONB type for efficient storage
);

-- Inserting example data
INSERT INTO entities (id, name, feature_vector)
VALUES
(1, 'entity1', ROW_TO_JSON(ROW(1.0, 2.0, 3.0))),
(2, 'entity2', ROW_TO_JSON(ROW(4.0, 5.0, 6.0)));

-- Query to retrieve a per-entity feature vector
SELECT
    id,
    jsonb_agg(feature) as feature_vector
FROM (
    SELECT
        id,
        (1, 2, 3) AS feature -- Example features for each entity
    FROM entities
) subquery
GROUP BY id;
```

