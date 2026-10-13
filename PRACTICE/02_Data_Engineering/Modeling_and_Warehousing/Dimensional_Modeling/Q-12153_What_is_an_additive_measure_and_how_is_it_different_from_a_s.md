---
type: "practice"
difficulty: "Deep"
---

# What is an additive measure, and how is it different from a semi-additive and a non-additive measure?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Additive and Allocated Measures](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Additive_and_Allocated_Measures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To begin with, in data modeling, measures are categorized based on their behavior when aggregated over time or across dimensions. An additive measure can be summed up without any issue; it's simple and straightforward. A semi-additive measure has more complex aggregation rules depending on the dimension being aggregated. And a non-additive measure is not aggregatable at all in certain contexts. Today, we'll explore these concepts through the lens of data modeling with star and snowflake schemas.

**Comparison:**

      |region_name|
      |product_name| price

**Code:**

```sql
-- Example tables for illustration
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    date DATE,
    region_id INT,
    product_id INT,
    quantity_sold INT,
    revenue DECIMAL(10, 2)
);

CREATE TABLE regions (
    region_id INT PRIMARY KEY,
    region_name VARCHAR(50)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10, 2)
);
```

**Example input / output:**

```text
Input:
- sales with date dimension (time), region_id (space), product_id

Output:
- Additive measure: quantity_sold and revenue are additive over time but not necessarily space or product.
- Semi-additive measure: revenue might be additive across products within a region, but not across regions in a single product.
- Non-additive measure: profit would not be additive at all if it depends on specific factors like seasonality or marketing campaigns that cannot be aggregated.
```

**Data:**

```json
{
  "sales": [
    {"sale_id": 1, "date": "2023-01-01", "region_id": 1, "product_id": 101, "quantity_sold": 50, "revenue": 4000.00},
    {"sale_id": 2, "date": "2023-01-01", "region_id": 2, "product_id": 101, "quantity_sold": 75, "revenue": 6000.00}
  ],
  "regions": [
    {"region_id": 1, "region_name": "North"},
    {"region_id": 2, "region_name": "South"}
  ],
  "products": [
    {"product_id": 101, "product_name": "Laptop", "price": 80.00},
    {"product_id": 102, "product_name": "Monitor", "price": 50.00}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Additive Measure
8. Is the measure time-related?
9. Sum over time
10. Does the space (region) affectaggregation?
11. Aggregation depends onregion
12. Simple sum across products
13. Semi-Additive Measure
14. Is the measureproduct-related?
15. Sum within a product, notcross-products
16. Sum within regions, but notacross all regions
17. Non-Additive Measure
18. Cannot be aggregatedregardless of context

