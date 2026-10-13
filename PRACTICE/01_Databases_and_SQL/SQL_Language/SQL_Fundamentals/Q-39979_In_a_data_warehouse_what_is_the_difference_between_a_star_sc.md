---
type: "practice"
difficulty: "Basic"
---

# In a data warehouse, what is the difference between a star schema and a snowflake schema, and how does each affect your JOIN queries?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Joining Type 2 Dimensions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Star and snowflake schemas are two common dimensional modeling techniques used in data warehouses. A star schema involves simpler fact tables surrounded by dimension tables, while a snowflake schema involves normalized dimension tables that can be more complex but offer better scalability.

The core difference lies in the level of normalization within the dimensions. In a star schema, dimension tables are fully denormalized and directly joined with the fact table, resulting in fewer JOIN operations but potentially larger storage requirements. Conversely, in a snowflake schema, some or all dimensions are normalized, leading to more complex join patterns as these smaller tables need to be combined.

To illustrate this concept:

Star Schema: Fact table ( F ) is directly joined with dimension tables ( D_1 ), ( D_2 ), and ( D_3 ).
Snowflake Schema: Dimension tables are further split into normalized sub-tables, for example, a Product dimension could be split into Category, Brand, and Model.

This triage clarifies that the question is probing your understanding of dimensional modeling, normalization levels, and their impact on query performance.

**Comparison:**

| Star Schema | Snowflake Schema |
|-------------|------------------|
| Fewer joins, more storage | More joins, less storage but better scalability |
| Simpler queries | Complex queries due to multiple joins |

**Code:**

```sql
-- Example Star Schema Join Query
SELECT f.date, p.category, p.brand, SUM(f.quantity)
FROM fact_sales f
JOIN dim_date d ON f.date_id = d.id
JOIN dim_product p ON f.product_id = p.id
GROUP BY f.date, p.category, p.brand;

-- Example Snowflake Schema Join Query
SELECT f.date, cat.name AS category_name, b.name AS brand_name, SUM(f.quantity)
FROM fact_sales f
JOIN dim_date d ON f.date_id = d.id
JOIN (dim_product p JOIN dim_category c ON p.category_id = c.id) ON f.product_id = p.id
JOIN (dim_brand b JOIN dim_category c2 ON b.category_id = c2.id) ON p.brand_id = b.id
GROUP BY f.date, cat.name, b.name;
```

**Example input / output:**

```

Star Schema Example Input & Output:

PLAINTEXT
Copy
Input:
Fact Sales Table: fact_sales (date_id, product_id, quantity)
Dimension Date Table: dim_date (id, date)
Dimension Product Table: dim_product (id, product_id, category_id, brand_id)

Output:
[
  {"date": "2023-10-01", "category_name": "Electronics", "brand_name": "Brand A", "quantity": 5},
  {"date": "2023-10-02", "category_name": "Electronics", "brand_name": "Brand B", "quantity": 3}
]

Snowflake Schema Example Input & Output:

PLAINTEXT
Copy
Input:
Fact Sales Table: fact_sales (date_id, product_id, quantity)
Dimension Date Table: dim_date (id, date)
Dimension Product Table: dim_product (id, product_id, category_id, brand_id),
                         dim_category (id, name),
                         dim_brand (id, name)

Output:
[
  {"date": "2023-10-01", "category_name": "Electronics", "brand_name": "Brand A", "quantity": 5},
  {"date": "2023-10-02", "category_name": "Electronics", "brand_name": "Brand B", "quantity": 3}
]
```

**Data:**

```

Fact Table: Contains measurements of facts and foreign keys to dimension tables.

JSON
Copy
{
  "fact_sales": [
    {"date_id": 101, "product_id": 201, "quantity": 5},
    {"date_id": 102, "product_id": 202, "quantity": 3}
  ]
}

Dimension Table: Contains descriptive information.

JSON
Copy
{
  "dim_date": [
    {"id": 101, "date": "2023-10-01"},
    {"id": 102, "date": "2023-10-02"}
  ],
  "dim_product": [
    {"id": 201, "product_id": 201, "category_id": 101, "brand_id": 101},
    {"id": 202, "product_id": 202, "category_id": 101, "brand_id": 102}
  ],
  "dim_category": [
    {"id": 101, "name": "Electronics"}
  ],
  "dim_brand": [
    {"id": 101, "name": "Brand A"},
    {"id": 102, "name": "Brand B"}
  ]
}
```

**Flow:**

1. SnowflakeSchema
2. StarSchema
3. Star Schema
4. Fact Table
5. Category, Brand
6. Dimension Table
7. Snowflake Schema
8. Fact Table
9. Category
10. Brand

