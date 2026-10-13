---
type: "practice"
difficulty: "Intermediate"
---

# Why has ELT become more common than ETL now that cloud warehouses have cheap compute and storage?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Late Arriving Dimensions](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In recent years, there has been a significant shift from ETL (Extract, Transform, Load) to ELT (Extract, Load, Transform). This change is largely due to advancements in cloud infrastructure that have made cheap compute and storage more accessible than ever before. In this answer, I will explain why ELT has become more common by discussing the key benefits it brings, especially within a cloud environment.

#### Triaging the Question

The question asks about the prevalence of ELT over ETL in light of improved cloud infrastructure. The core idea here is to understand the operational and cost advantages that make ELT preferable for modern data engineering practices, particularly when leveraging cloud warehouses.

#### First-Principles Solution

ETL traditionally involves three steps: extracting data from sources, transforming it into a consistent format, and then loading it into a target system. In contrast, ELT skips the transformation step during extraction and performs transformations post-load. This change is driven by several factors:

Cost Efficiency: Cloud providers often offer highly scalable storage and compute resources at lower costs compared to on-premises solutions.
Flexibility and Agility: ELT allows for more flexible data processing pipelines, enabling real-time or near-real-time transformations without the need for upfront schema design.
Data Quality Improvements: Post-load transformations can be optimized using powerful cloud-based analytics tools, which might not be as easily accessible during ETL.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, ELT has become more common because it leverages the cost and flexibility benefits of modern cloud infrastructure. The main advantage is that data can be stored in a consistent format post-load, allowing for better use of advanced transformation tools and analytics platforms available in the cloud.

#### Common mistakes
include underestimating the importance of upfront schema design during ETL or overlooking the potential costs associated with excessive transformations performed after loading data into a warehouse.

Closing

For further exploration, one might consider discussing the trade-offs between ETL and ELT when dealing with large-scale data ingestion pipelines, the impact on data lineage and governance, and how to optimize cloud storage and compute resources for efficient ELT workflows.

**Comparison:**

```
ID	Value
1	5
2	8
Transformed Values (ETL)	Transformed Values (ELT)
15	[15, 18]
```

**Code:**

```python
# Example of a simple transformation step in an ETL pipeline using Python
def transform_data(data):
    # Simple transformation logic
    transformed_data = [x['value'] + 10 for x in data]
    return transformed_data
```

**Example input / output:**

```

Input:

PYTHON
Copy
data = [{'id': 1, 'value': 5}, {'id': 2, 'value': 8}]
transformed_data = transform_data(data)

Output:

PYTHON
Copy
[15, 18]
```

**Data:**

```json
{
  "data": [
    { "id": 1, "value": 5 },
    { "id": 2, "value": 8 }
  ]
}
```

**Flow:**

1. Raw Data
2. Transformed Data
3. Adjusted Data
4. Transformed Data
5. Data Source
6. Extract
7. Cloud Warehouse
8. Transform
9. Target System
10. ETL Process
11. Load
12. Transform
13. Target System
14. ELT Process
15. Load
16. Transform
17. Target System

