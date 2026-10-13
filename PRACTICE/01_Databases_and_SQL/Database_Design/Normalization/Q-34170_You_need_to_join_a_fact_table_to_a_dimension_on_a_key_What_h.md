---
type: "practice"
difficulty: "Intermediate"
---

# You need to join a fact table to a dimension on a key. What happens if that key is missing in the dimension?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Referential Integrity Checks](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Referential_Integrity_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When joining a fact table to a dimension using a key, and that key is missing in the dimension, several outcomes can occur depending on how the database handles such situations. In SQL databases, if the join condition is set with an INNER JOIN, rows from the fact table without matching keys in the dimension will be excluded from the result. If you use LEFT JOIN or FULL OUTER JOIN, these unmatched rows from the fact table are retained but will have null values for columns of the dimension.

The question probing here involves understanding how SQL joins work and their implications on data integrity, as well as how to handle missing keys in a production environment.

**Code:**

```sql
-- Example using INNER JOIN
SELECT *
FROM fact_table ft
INNER JOIN dim_table dt ON ft.key = dt.key;

-- Example using LEFT JOIN
SELECT *
FROM fact_table ft
LEFT JOIN dim_table dt ON ft.key = dt.key;
```

**Example input / output:**

```

Example input -> output:

SQL
Copy
-- INNER JOIN example
SELECT *
FROM fact_table ft
INNER JOIN dim_table dt ON ft.key = dt.key;

/* Result:
id | key | value | id | key
---|-----|-------|----|-----
1  | k1  | 100   | 1  | k1 */
SQL
Copy
-- LEFT JOIN example
SELECT *
FROM fact_table ft
LEFT JOIN dim_table dt ON ft.key = dt.key;

/* Result:
id | key | value | id | key
---|-----|-------|----|-----
1  | k1  | 100   | 1  | k1
2  | k2  | 200   | NULL | NULL */
```

**Data:**

```

Sample data schemas:

JSON
Copy
{
  "fact_table": {
    "id": [1, 2],
    "key": ["k1", "k2"],
    "value": [100, 200]
  },
  "dim_table": {
    "id": [1, 3],
    "key": ["k1", "k3"]
  }
}
```

