---
type: "practice"
difficulty: "Deep"
---

# How would you use the catalog to power impact analysis before a schema change ships — telling the engineer 'this drop-column will break 12 dashboards and 3 ML models'? What has to be true about lineage coverage for that to be trustworthy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Catalog Consistency Model](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Catalog_Consistency_Model.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To use a data catalog effectively for impact analysis before making schema changes, you need a comprehensive lineage model that traces how each column impacts various downstream processes such as dashboards and ML models. The goal is to predict potential disruptions accurately so engineers can make informed decisions. Here’s a step-by-step approach from first principles:

TL;DR: Establishing detailed lineage in your data catalog ensures you can trace dependencies, making it possible to identify which dashboards or ML models might be impacted by a schema change.

Understanding the Question & Probing: The question revolves around leveraging the metadata stored within a data catalog to perform impact analysis. Specifically, how would we determine that a 'drop-column' operation could break 12 dashboards and 3 ML models? This requires robust lineage coverage that captures all dependencies on each column.

Step-by-Step Solution:

Lineage Model: Ensure your data catalog has a well-defined lineage model. Each column should have metadata about its usage across different applications (e.g., databases, analytics tools, ML workflows). Tools like Apache Atlas or AWS Glue can help in building such a comprehensive lineage.
Impact Analysis Workflow: Develop an automated pipeline that uses the lineage information to simulate schema changes and analyze their impact. This involves:
Column Removal Simulation: Temporarily mark the column as removed (or use a flag) without actually changing it in production.
Dependency Detection: Use the lineage model to identify all downstream processes affected by this change.
Impact Assessment: For each identified downstream process, assess whether it would be impacted based on its dependency on the column.

#### Trade-offs
and Considerations:

Performance vs. Accuracy: Building a detailed lineage model is resource-intensive but crucial for accurate impact analysis. Balance between the granularity of dependencies and the performance overhead.
Automation & Scalability: Automate the process to handle frequent schema changes efficiently, ensuring that the pipeline can scale with increasing data complexity.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

For this approach to be trustworthy, lineage coverage must be comprehensive and regularly updated. Any gaps in the lineage model could lead to false negatives or false positives.

#### Common mistakes
include underestimating the number of downstream dependencies or failing to automate the process, leading to manual checks that are error-prone.

**Comparison:**

```
Column	Dependencies
customer_id	dashboard_sales, ml_customer_segmentation
product_name	dashboard_sales
```

**Code:**

```python
# Example code snippet for automated impact analysis pipeline

import pandas as pd
from lineage_model import get_downstream_dependencies

def simulate_drop_column(column_name):
    # Mark column as removed in production database (simulated)
    mark_column_as_removed_in_production_db(column_name)

    # Simulate schema change and detect dependencies
    downstream_dependencies = get_downstream_dependencies(column_name)

    # Assess impact on dashboards and ML models
    impacted_components = assess_impact(downstream_dependencies)

    return impacted_components

def assess_impact(dependencies):
    # Check which components depend on the column
    impacted_dashboards = [dashboard for dashboard in dependencies if 'dashboard' in dashboard]
    impacted_ml_models = [model for model in dependencies if 'ml_model' in model]

    return {"dashboards": impacted_dashboards, "ml_models": impacted_ml_models}

# Example usage
impacted_components = simulate_drop_column("customer_id")
print(impacted_components)
```

**Example input / output:**

```text
Input:
- Simulate removing column 'customer_id'

Output:
- Impacted components:
  - Dashboards: ['dashboard_sales']
  - ML models: ['ml_customer_segmentation']
```

**Data:**

```json
{
  "lineage_model": {
    "columns": [
      {
        "name": "customer_id",
        "dependencies": ["dashboard_sales", "ml_customer_segmentation"]
      },
      {
        "name": "product_name",
        "dependencies": ["dashboard_sales"]
      }
    ],
    "applications": [
      {
        "type": "dashboard",
        "name": "dashboard_sales"
      },
      {
        "type": "ml_model",
        "name": "ml_customer_segmentation"
      }
    ]
  }
}
```

**Flow:**

1. Mark Column as Removed
2. Get DownstreamDependencies
3. Automate Impact Assessment
4. Assess Dashboards & MLModels

