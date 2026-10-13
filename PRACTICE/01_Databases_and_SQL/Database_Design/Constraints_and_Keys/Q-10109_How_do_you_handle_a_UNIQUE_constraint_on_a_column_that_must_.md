---
type: "practice"
difficulty: "Deep"
---

# How do you handle a UNIQUE constraint on a column that must be unique only within a soft-partition (per-year) — partial index, generated column, or partitioned unique?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Unique and Deferred Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the scenario where a UNIQUE constraint needs to be enforced only within a specific partition (e.g., per year), we have several options including using a partial index, a generated column, and partitioning techniques. Each approach has its trade-offs in terms of performance, complexity, and manageability.

**Comparison:**

```
Rendering diagram…
```

**Code:**

```sql
-- Example of a generated column to store the year
ALTER TABLE sales ADD COLUMN sale_year INT AS (YEAR(sale_date)) STORED;

-- Partial index based on the year partition
CREATE INDEX idx_unique_sales_year ON sales (product_id) WHERE YEAR(sale_date) = 2023;

-- Partitioned table with unique constraint per partition
CREATE TABLE sales (
    id INT,
    product_id INT,
    sale_date DATE,
    PRIMARY KEY (id, sale_year),
    UNIQUE (product_id) WITH PARTITION
);
```

**Example input / output:**

```text
-- Inserting valid data into the partitioned table
INSERT INTO sales (id, product_id, sale_date) VALUES (1, 101, '2023-01-01');
-- This will succeed as it falls within the unique constraint for 2023

-- Attempting to insert duplicate data in the same year
INSERT INTO sales (id, product_id, sale_date) VALUES (2, 101, '2023-01-05');
-- This will fail due to the unique constraint on product_id within 2023

-- Inserting valid data across different years
INSERT INTO sales (id, product_id, sale_date) VALUES (3, 102, '2024-01-01');
-- This will succeed as it is in a separate partition for the year 2024
```

**Data:**

```json
{
  "sales": [
    {"id": 1, "product_id": 101, "sale_date": "2023-01-01"},
    {"id": 2, "product_id": 102, "sale_date": "2023-02-01"}
  ]
}
```

**Flow:**

1. Rendering diagram…

