---
type: "practice"
difficulty: "Basic"
---

# In your own words, what problem is dimensional modeling trying to solve for analytics?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Special Dimension Types](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Dimensional modeling is a key technique in database design aimed at optimizing data storage and query performance for business intelligence and analytics. The main problems it addresses are:

Handling Large Volumes of Data: Businesses often need to store massive amounts of historical data, which can become unwieldy if not properly organized.
Supporting Complex Queries Efficiently: Users frequently run complex analytical queries that require fast retrieval of summarized and aggregated data.

Dimensional modeling achieves these goals through a specific schema design: the Star Schema and Snowflake Schema.

**Example input / output:**

```text
# Example Input-Output for a Star Schema Query

**Input:**
- SQL query to find total sales by month and product category.

**Output:**
| Month | Category | Total Sales |
|-------|----------|-------------|
| Jan   | Electronics | 5000        |
| Jan   | Clothing   | 3000        |
| Feb   | Electronics | 6000        |
| Feb   | Clothing   | 4000        |
```

**Data:**

```yaml
# Example Data Model Schemas

Star Schema:
- Fact Table: sales (date, product_id, store_id, amount)
- Dimension Tables:
  - date_dim: (date_key, year, month, day)
  - product_dim: (product_id, product_name, category)
  - store_dim: (store_id, store_name, location)

Snowflake Schema:
- Fact Table: sales (date_key, product_key, store_key, amount)
- Dimension Tables:
  - date_dim: (date_key, year, month, day)
  - product_dim: (product_id, product_name, category, brand)
  - store_dim: (store_id, store_name, location, region)
```

