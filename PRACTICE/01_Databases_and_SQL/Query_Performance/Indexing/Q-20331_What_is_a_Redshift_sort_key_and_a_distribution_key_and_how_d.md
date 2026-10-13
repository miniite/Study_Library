---
type: "practice"
difficulty: "Deep"
---

# What is a Redshift sort key and a distribution key, and how do they play the role that indexes play in a row-store?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Random Keys and Index Locality](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Random_Keys_and_Index_Locality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Redshift sort keys and distribution keys are crucial concepts for optimizing query performance in Amazon Redshift, which is a columnar storage database designed for large-scale data warehousing. They serve roles similar to indexes in row-stored databases but have distinct characteristics tailored for columnar storage.

In this context:

Sort Keys: Similar to primary key or clustered index in row-store databases, sort keys are used to order the data within a Redshift segment (a collection of rows).
Distribution Keys: Define how the data is distributed across nodes in a Redshift cluster, ensuring that queries can be efficiently executed by minimizing cross-node operations.

Let's dive into each concept and understand their roles step-by-step from first principles.

**Comparison:**

| Column Name | Type      |
|-------------|-----------|
| customer_id | INTEGER   |
| order_date  | DATE      |
| product_id  | VARCHAR(20) |
| quantity    | INTEGER   |

**Code:**

```python
# Example code to demonstrate basic understanding of sort keys and distribution keys in Python (simplified)
def create_distribution_key(table_name, column_names):
    return f"CREATE TABLE {table_name} SORTKEY ({', '.join(column_names)}) DISTKEY ('{column_names[0]}');"

distribution_key = create_distribution_key("sales_data", ["customer_id", "order_date"])
print(distribution_key)
```

**Example input / output:**

```text
Input:
- Table name: sales_data
- Columns: customer_id, order_date

Output:
CREATE TABLE sales_data SORTKEY (customer_id, order_date) DISTKEY ('customer_id');
```

**Data:**

```json
{
  "table": {
    "name": "sales_data",
    "columns": [
      {"name": "customer_id", "type": "INTEGER"},
      {"name": "order_date", "type": "DATE"},
      {"name": "product_id", "type": "VARCHAR(20)"},
      {"name": "quantity", "type": "INTEGER"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data is Loaded into Redshift
6. Are Sort Keys Used?
7. Data is Sorted by Sort Key
8. Query Performance ImprovedDue to Ordered Data
9. Are Distribution Keys Used?
10. Data is Distributed AcrossNodes Based on DistributionKey
11. Minimizes Cross-NodeOperations, Improves QueryPerformance
12. Data Not Optimized forQueries

