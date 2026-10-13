---
type: "practice"
difficulty: "Intermediate"
---

# What's your strategy for governing dbt exposures so BI dashboards are first-class nodes in your lineage graph?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Ownership](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a data-driven organization, ensuring that BI dashboards rely on well-governed and lineage-capturing dbt models is crucial. My strategy focuses on three key areas: establishing clear policies, implementing robust automation, and ensuring continuous monitoring.

#### Triaging the Question

The question asks about a governance strategy for dbt exposures, particularly in ensuring that BI dashboards are considered first-class nodes in the lineage graph. The interviewer is probing your understanding of how to integrate business intelligence (BI) into data engineering practices effectively. You'll need to demonstrate knowledge of policies, automation tools, and monitoring mechanisms.

#### Assumptions
Tooling: We use dbt for data transformation.
Lineage Graph: This refers to a graph that traces the lineage of data through various transformations and models.
BI Dashboards: These are used by business analysts and stakeholders to visualize key performance indicators (KPIs) derived from transformed data.
Step-by-Step Solution

Policy Definition:

Establish clear policies on how dbt models should be structured and named to align with the organization's data governance standards.
Ensure that all exposure models are tagged appropriately, making it easy to trace their lineage through BI dashboards.

Automation Setup:

Integrate dbt with a metadata management tool like Great Expectations or DBT-LABS for automated testing and validation of exposures.
Use dbt hooks or schedules to run lineage tracking scripts automatically whenever models are updated.

Monitoring and Auditing:

Implement continuous monitoring using tools like dbt's built-in reporting features to track changes in exposure models.
Regularly audit the lineage graph to ensure all relevant dependencies are captured accurately, especially for critical BI dashboards.
Detailed Implementation

Policy Definition

Define naming conventions: e.g., use prefixes or suffixes to indicate if a model is an exposure (exposure__).
Use tags to classify models based on their business context and data sensitivity.
SQL
-- Example of tagging a dbt model
{{ config(
  tags=['finance', 'critical'],
  materialized='table'
) }}

Automation Setup

Create a dbt project that includes lineage tracking models. These models will capture the dependencies between dbt nodes and their outputs.
SQL
-- Example of a simple lineage tracking model
{{ config(
  materialized='table',
  tags=['lineage']
) }}
select
  exposure_name,
  dependent_model,
  transformation_time
from
  dbt_yourteam.exposure__customer_segmentation
join
  dbt_yourteam.insights__dashboard_metrics
on
  dbt_yourteam.exposure__customer_segmentation.customer_id = dbt_yourteam.insights__dashboard_metrics.customer_id;
Configure dbt hooks to run lineage tracking scripts before and after model runs.
YAML
-- Example of a hook in dbt_project.yml
hooks:
  pre-hook:
    'select * from dbt_yourteam.lineage_tracking_model'
  post-hook:
    'insert into dbt_yourteam.alerts (model, status) values ("{{this.name}}", "success")'

Monitoring and Auditing

Set up automated alerts for lineage changes that affect critical BI dashboards.
Use a dashboard tool like Looker or PowerBI to visualize the lineage graph and highlight any discrepancies.

#### Closing Summary

By implementing clear policies, leveraging automation tools, and ensuring continuous monitoring, you can effectively govern dbt exposures and maintain first-class nodes in your lineage graph for BI dashboards. This approach not only enhances data quality but also ensures compliance with organizational data governance standards.

#### Common Mistakes/Follow-Ups
Ignoring manual validation: Automated systems are prone to errors; regular manual checks are necessary.
Overlooking edge cases: Ensure that all possible scenarios, such as data inconsistencies or model dependencies, are considered in the lineage tracking mechanism.
Failure modes: Discuss potential failure points, like issues with dbt runs failing and not updating the lineage graph.

**Comparison:**

| Exposures | Dependent Models |
|-----------|------------------|
| customer_segmentation | dashboard_metrics |

**Code:**

```yaml
# Example of a dbt_project.yml hook configuration
hooks:
  pre-hook:
    - 'select * from dbt_yourteam.lineage_tracking_model'
  post-hook:
    - 'insert into dbt_yourteam.alerts (model, status) values ("{{this.name}}", "success")'
```

**Example input / output:**

```
Input: Running a dbt model exposure__customer_segmentation.
Output: lineage_tracking_model is updated with new dependencies, and alerts are sent for any critical changes.
Dry-run:
SQL
Copy
select * from dbt_yourteam.exposure__customer_segmentation;
insert into dbt_yourteam.alerts (model, status) values ("customer_segmentation", "success");
```

**Data:**

```json
{
  "exposure_name": "customer_segmentation",
  "dependent_model": "dashboard_metrics",
  "transformation_time": "2023-10-01T14:59:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Initiate Model Run
4. Run Lineage Tracking Script
5. Is Model Critical?
6. Send Alerts
7. Update Metadata Tables
8. Record Transformation Time
9. Log Changes for Auditing

