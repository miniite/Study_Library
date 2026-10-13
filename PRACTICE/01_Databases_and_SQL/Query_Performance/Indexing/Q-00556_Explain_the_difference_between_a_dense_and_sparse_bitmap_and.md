---
type: "practice"
difficulty: "Expert"
---

# Explain the difference between a dense and sparse bitmap and how the optimizer estimates cost across them.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bitmap Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database indexing, bitmaps are used to efficiently represent the distribution of values in columns or combinations of columns. The key difference lies in their density (i.e., the number of bits set) and how an optimizer estimates costs involving these structures.

A dense bitmap has a high ratio of '1' bits, meaning it is nearly fully populated with 1s across its length, indicating that most rows have this combination of column values. Conversely, a sparse bitmap has few '1' bits, which means only a small portion of rows match the given conditions in the query.

When an optimizer evaluates queries using these bitmaps, the cost estimation depends on several factors, including the density and how effectively the bitmap reduces the number of rows to scan.

To understand this better, let's break down the concepts from first principles.

**Code:**

```sql
-- Example schema for explanation
CREATE TABLE products (
    product_id INT,
    category_id INT,
    price DECIMAL(10,2),
    CONSTRAINT pk PRIMARY KEY (product_id)
);

-- Insert some sample data
INSERT INTO products VALUES
(1, 1, 50.00),
(2, 1, 60.00),
(3, 2, 70.00),
(4, 2, 80.00),
(5, 3, 90.00);
```

**Example input / output:**

```text
### Dense Bitmap Example
Consider a query like `SELECT * FROM products WHERE category_id = 1`. Here's how the bitmap might look:
- **Sparse Bitmap**: `0b100` (only one row out of four matches).
- **Dense Bitmap**: `0b111` (three rows out of four match).

### Cost Estimation
The optimizer would estimate a lower cost for the dense bitmap because it suggests more relevant rows, reducing the need to scan the entire table. Sparse bitmaps imply fewer matching rows, increasing the estimated cost.

| Column | Dense Bitmap | Sparse Bitmap |
|--------|--------------|---------------|
| category_id = 1 | `0b111` (3 matches) | `0b100` (1 match) |

### Cost Calculation
- **Dense Bitmap**: Cost ≈ Cost to read 3 rows.
- **Sparse Bitmap**: Cost ≈ Cost to read all rows + Cost to skip non-matching rows.

In summary, the optimizer uses bitmap density to refine its cost estimates. A dense bitmap leads to lower estimated costs due to higher matching row counts, while sparse bitmaps increase costs by indicating fewer relevant rows.
```

**Data:**

```json
[
    {"product_id": 1, "category_id": 1},
    {"product_id": 2, "category_id": 1},
    {"product_id": 3, "category_id": 2},
    {"product_id": 4, "category_id": 2},
    {"product_id": 5, "category_id": 3}
]
```

