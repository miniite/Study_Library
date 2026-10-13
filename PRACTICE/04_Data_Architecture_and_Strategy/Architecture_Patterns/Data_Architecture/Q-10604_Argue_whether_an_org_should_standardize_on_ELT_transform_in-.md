---
type: "practice"
difficulty: "Deep"
---

# Argue whether an org should standardize on ELT (transform in-warehouse) or ETL (transform before load) as its paradigm. What does each imply for cost, governance, and where data engineering talent must sit?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse as System of Record](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between ELT (Extract, Load, Transform in-warehouse) and ETL (Extract, Transform, Load), organizations must weigh several factors including cost, governance, and the placement of data engineering talent. The choice impacts how data flows through different systems, the complexity of operations, and the overall architecture of their data stack.

Firstly, cost is a critical factor:

ELT: This approach reduces ETL processing costs since transformations are performed in more powerful warehouse environments with more resources. However, it increases storage costs because raw data must be stored before any transformation.
ETL: On the other hand, ETL reduces storage costs as the transformed data is immediately loaded into the warehouse. However, it can lead to increased computational and processing costs due to the need for complex ETL jobs.

Secondly, governance:

ELT: Provides more flexibility in how transformations are applied and managed. It allows for easier experimentation with different transformation methods without impacting existing systems.
ETL: Offers a more controlled environment as all transformations are predefined and occur before loading into the warehouse. This can simplify governance but may limit agility.

Finally, the placement of data engineering talent:

ELT: Requires data engineers to work closely with the warehouse and analytics tools, often in the same environment where transformations occur.
ETL: Places more emphasis on having strong ETL expertise within a separate team or department that interfaces between raw data sources and the warehouse.

**Comparison:**

| Source Type       | ELT                   | ETL                    |
|------------------|-----------------------|------------------------|
| **Cost**         | Reduced processing    | Increased storage      |
|                  | costs (warehouse)     |                        |
|                  | Increased storage     | Reduced processing     |
|                  | costs                 |                        |
| **Governance**   | More flexible         | Easier to govern       |
|                  | experimentation       |                       |
|                  |                      |                        |
| **Talent Placement** | Data engineers in    | ETL team separate      |
|                     | warehouse             | from other teams      |

**Code:**

```text
# Example of an ELT workflow
def load_data():
    extract_from_sources()
    transform_in_warehouse()
    load_into_analytics()

# Example of an ETL workflow
def transform_and_load():
    extract_from_sources()
    perform_complex_transformations()
    load_into_target_store()
```

**Example input / output:**

| Source Type | Transform in Warehouse? | Storage Cost Increase | Processing Cost Decrease |
|-------------|-----------------------|----------------------|-------------------------|
| Sales Data  | No                    | Yes                  | No                      |
| Customer Feedback | Yes            | Yes                  | Yes                     |

**Data:**

```json
{
  "data": [
    {
      "source": "Sales Database",
      "transform_in_warehouse": false,
      "storage_cost_increase": true,
      "processing_cost_decrease": false
    },
    {
      "source": "Customer Feedback",
      "transform_in_warehouse": true,
      "storage_cost_increase": true,
      "processing_cost_decrease": true
    }
  ]
}
```

**Flow:**

1. Data Extraction
2. Transform in Warehouse
3. Load into Analytics
4. Increased Storage Costs
5. Reduced Processing Costs
6. Transform Before Load
7. Load into Target Store
8. Increased Processing Costs
9. Reduced Storage Costs
10. Cheap Warehouse Resources
11. Flexible Transformations
12. Complex ETL Jobs
13. Controlled Environment

