---
type: "practice"
difficulty: "Intermediate"
---

# Explain why you'd choose ELT into a modern cloud warehouse over classic ETL when the OLTP source dumps are large and transformations change frequently.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Separating OLTP and Analytics](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When choosing between ETL (Extract, Transform, Load) and ELT (Extract, Load, Transform), in a modern cloud warehouse context, especially with large OLTP (Online Transaction Processing) source dumps and frequently changing transformations, the key advantage of ELT is its ability to leverage the power and flexibility of the cloud data warehouse. This approach allows for more agile and efficient processing, which can be particularly beneficial when dealing with complex and evolving transformation needs.

#### Triage
What's Being Probed

The question is essentially asking why you would opt for ELT into a modern cloud warehouse over traditional ETL, given that the OLTP source dumps are large and transformations change frequently. The interviewer will likely probe your understanding of data processing workflows, storage vs. compute costs in the cloud, and how different approaches affect development cycles.

#### Assumptions
Large Source Data: The OLTP system generates significant amounts of data.
Frequent Transformations: The transformation logic is dynamic and changes frequently.
Modern Cloud Warehouse: Assume AWS Redshift, Google BigQuery, or Snowflake as the target warehouse.
Step-by-Step Solution
Why Not ETL?

Compute Costs:

In traditional ETL pipelines, transformations are done on-premises or in a staging environment before loading into the data warehouse.
This approach can lead to higher compute costs and longer processing times as large datasets are moved and transformed.

Latency:

Transformations performed outside of the data warehouse can introduce latency, delaying when insights become available.

Inflexibility:

With frequent changes in transformations, ETL pipelines need constant maintenance and reconfiguration to reflect these changes, leading to increased operational overhead.
Why Choose ELT

Cost-Efficiency:

Modern cloud warehouses typically offer more cost-effective compute resources when transformations are performed within the warehouse.
In ELT, you can leverage the warehouse's powerful processing capabilities without having to build and maintain complex ETL jobs on-premises.

Reduced Latency:

By performing transformations directly in the data warehouse, insights can be derived more quickly and efficiently.
This is particularly useful for real-time or near-real-time analytics where speed of insight is critical.

Flexibility and Agility:

ELT allows for simpler development cycles as you don't need to maintain complex ETL jobs.
You can easily tweak transformations in the warehouse, reducing maintenance overhead and enabling faster iterations on analytical queries.

Scalability:

Modern cloud warehouses are highly scalable, making it easier to handle large datasets and frequent changes without significant performance degradation.

Data Quality and Governance:

Storing raw data first allows for better data quality checks and governance, as transformations can be applied selectively based on need.

#### Closing Summary

In summary, ELT into a modern cloud warehouse is the preferred approach when dealing with large OLTP source dumps and frequently changing transformations. The cost-efficiency, reduced latency, flexibility, and scalability of the cloud warehouses make them an attractive choice for modern data processing needs. This approach also simplifies development cycles, enabling faster iterations and improved agility in responding to business requirements.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the compute costs associated with performing transformations in the warehouse.

#### Follow-Up
Discussing scenarios where ETL might be more appropriate, such as when dealing with complex data flows that require specialized processing or when compliance regulations mandate strict control over data transformation.

**Comparison:**

| Transformation | ETL | ELT |
|----------------|-----|-----|
| Cost           | High | Low  |
| Latency        | High | Low  |
| Flexibility    | Low  | High |
| Scalability    | Medium | High |

**Code:**

```python
# Example of a simple ELT pipeline using Python and Snowflake
import snowflake.connector

def load_data_into_snowflake(df):
    conn = snowflake.connector.connect(
        user='user',
        password='password',
        account='account'
    )
    cursor = conn.cursor()

    # Load raw data into the warehouse
    with open('raw_data.csv', 'rb') as file:
        cursor.copy_to_table('RAW_DATA_TABLE', file)

    # Perform transformation (simplified example)
    transformed_data = transform_data(df)
    for row in transformed_data:
        insert_query = "INSERT INTO TRANSFORMED_DATA VALUES (%s, %s)"
        cursor.execute(insert_query, row)

    conn.commit()
    conn.close()

def transform_data(df):
    # Simple transformation logic
    return [(row['col1'], row['col2']) for index, row in df.iterrows()]
```

**Example input / output:**

```text
Input:
- Large OLTP source dumps with frequent schema changes.

Output:
- Raw data stored in the warehouse.
- Transformations applied within the warehouse.
- Faster and more flexible processing of complex transformations.
```

**Data:**

```json
{
  "raw_data_table": {
    "columns": ["id", "timestamp", "value"]
  },
  "transformed_data_table": {
    "columns": ["id", "calculated_value"]
  }
}
```

**Flow:**

1. OLTP Source
2. Extract
3. Raw Data to Warehouse
4. Transformations in Warehouse
5. Insights
6. Transformations OutsideWarehouse
7. Compute Costs High
8. Latency High
9. Maintenance Overhead

