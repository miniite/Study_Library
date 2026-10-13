---
type: "practice"
difficulty: "Expert"
---

# How do you catalog and govern derived/semantic-layer assets (metrics, dimensions defined in a metrics layer)? Should the metric definition itself be a first-class cataloged entity, and how does that relate to physical lineage?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cataloging External Data](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Cataloging_External_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively catalog and govern derived or semantic-layer assets such as metrics and dimensions, we need to ensure robust metadata management practices. At an architectural level, the metric definition should indeed be a first-class cataloged entity, which is critical for maintaining transparency and traceability throughout the data ecosystem.

TL;DR

The key idea here is that metric definitions should be fully cataloged alongside their physical lineage. This ensures that all stakeholders can understand how metrics are calculated, where they come from, and who owns them. The metadata should capture not just what the metrics are but also the processes used to derive them.

#### Triaging the Question

The question revolves around cataloging derived assets like metrics in a data catalog. Specifically, it asks about making sure that metric definitions themselves are treated as first-class entities. This involves understanding how these definitions are related to physical data sources and transformations. The interviewer is probing your knowledge of metadata governance, lineage tracking, and the importance of detailed metadata for maintaining trust in analytical assets.

#### Assumptions
We use a modern data catalog system that supports rich metadata.
Metrics and dimensions are derived from raw data through transformations or aggregation.
There is a need to govern how these metrics are used across different systems and teams.

#### First Principles Solution

Metadata Model: Define a robust metadata model where metric definitions include:

Name, description
Calculation logic (formula or SQL query)
Dependencies on raw data sources or other metrics
Owners, version control details
Timestamps for when the definition was last updated and approved

Physical Lineage: Capture the physical lineage by documenting each step of the transformation process:

Source tables and columns used in metric calculations
ETL processes that transform raw data into derived metrics
Intermediate tables or views where partial results are stored

Governance Policies: Implement policies around how metrics can be created, modified, and approved by relevant stakeholders (e.g., business analysts, data engineers). Ensure these policies are documented and enforced through metadata management tools.

Tooling Support: Use data catalog tools that support rich metadata capture and lineage tracking. Tools like Apache Atlas, AWS Glue Data Catalog, or Snowflake’s built-in metadata features can be leveraged to automate much of this process.

Usage Tracking: Track how metrics are used in reports, dashboards, and downstream systems. This helps identify potential issues early on and ensures that metrics remain relevant and accurate over time.

Change Management: Implement a change management workflow for metric definitions. Changes should be reviewed, approved by appropriate stakeholders before being deployed to production environments.

#### Closing Summary

In summary, cataloging metric definitions as first-class entities is crucial for effective metadata governance and ensuring the integrity of derived assets. By capturing detailed lineage and enforcing strict governance policies, we can build a robust data ecosystem that supports informed decision-making across teams.

#### Common Mistakes/Follow-Ups
Not Capturing Calculation Logic: Failing to store the precise calculation logic used in metrics can lead to inconsistencies.
Lack of Ownership and Version Control: Without clear ownership and version control, changes to metric definitions may not be properly tracked or approved.
Insufficient Lineage Tracking: Not capturing the full physical lineage can make it difficult to understand how data transformations impact final results.

For follow-ups, consider discussing specific tools and frameworks that support metadata governance, such as OpenMetadata or DataHub. Additionally, discuss best practices for integrating metadata management into existing data pipelines and ensuring continuous improvement in metadata quality.

**Comparison:**

| Metric Name | Calculation Logic | Owner(s)          | Last Updated |
|-------------|-------------------|------------------|--------------|
| CAC         | SUM(spend) / SUM(new_customers) | John Doe, Jane Smith | 2023-10-05   |

**Code:**

```python
# Example code snippet to capture metric definition metadata
class MetricDefinition:
    def __init__(self, name, description, calculation_logic, owners):
        self.name = name
        self.description = description
        self.calculation_logic = calculation_logic
        self.owners = owners

# Sample metric definition object
metric_def = MetricDefinition(
    name="Customer Acquisition Cost (CAC)",
    description="Cost of acquiring a new customer over a specific period",
    calculation_logic="SUM(spend) / SUM(new customers)",
    owners=["John Doe", "Jane Smith"]
)
```

**Example input / output:**

```text
Input:
Metric Definition Name: Customer Acquisition Cost (CAC)

Output:
{
  "name": "Customer Acquisition Cost (CAC)",
  "description": "Cost of acquiring a new customer over a specific period",
  "calculation_logic": "SUM(spend) / SUM(new_customers)",
  "owners": ["John Doe", "Jane Smith"],
  "dependencies": [
    {"table": "customer_events", "columns": ["spend", "new_customers"]}
  ],
  "lineage": {
    "steps": [
      {"source": "customer_events", "transformations": ["SUM(spend)"], "target": "monthly_spend"},
      {"source": "customer_events", "transformations": ["SUM(new_customers)"], "target": "new_customers_count"},
      {"source": ["monthly_spend", "new_customers_count"], "transformations": ["SUM(monthly_spend) / SUM(new_customers_count)"], "target": "CAC"}
    ],
    "tools_used": ["Apache Spark"]
  },
  "version_history": [
    {"version": 1, "date": "2023-10-01", "changes": ["Initial definition"]},
    {"version": 2, "date": "2023-10-05", "changes": ["Revised calculation logic"]}
  ]
}
```

**Data:**

```json
{
  "metric": {
    "name": "Customer Acquisition Cost (CAC)",
    "description": "Cost of acquiring a new customer over a specific period",
    "calculation_logic": "SUM(spend) / SUM(new customers)",
    "owners": ["John Doe", "Jane Smith"],
    "dependencies": [
      {"table": "customer_events", "columns": ["spend", "new_customers"]}
    ],
    "lineage": {
      "steps": [
        {"source": "customer_events", "transformations": ["SUM(spend)"], "target": "monthly_spend"},
        {"source": "customer_events", "transformations": ["SUM(new_customers)"], "target": "new_customers_count"},
        {"source": ["monthly_spend", "new_customers_count"], "transformations": ["SUM(monthly_spend) / SUM(new_customers_count)"], "target": "CAC"}
      ],
      "tools_used": ["Apache Spark"]
    },
    "version_history": [
      {"version": 1, "date": "2023-10-01", "changes": ["Initial definition"]},
      {"version": 2, "date": "2023-10-05", "changes": ["Revised calculation logic"]}
    ]
  }
}
```

**Flow:**

1. Data Source
2. ETL Process
3. Metric Calculation
4. Metric Storage in DataWarehouse
5. Metric Usage Tracking
6. Lineage Documentation
7. Change ManagementWorkflow
8. Ownership and VersionControl

