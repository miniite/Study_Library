---
type: "practice"
difficulty: "Deep"
---

# How do you prevent metric drift where the same KPI is computed three different ways across dashboards, using dbt's structure and governance features?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ dbt Models and Runs](<../../../../TOOLS/dbt/dbt_Core/dbt_Models_and_Runs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To prevent metric drift in a data warehouse environment, we need to ensure that key performance indicators (KPIs) are consistently calculated across multiple dashboards. Using dbt (data build tool), this can be achieved through rigorous governance and standardization of how metrics are defined, computed, and versioned. The core idea is to centralize the definition of these metrics in a single source of truth within your dbt project.

The question at its heart is about ensuring consistency and avoiding discrepancies when calculating KPIs across different dashboards or stakeholders. dbt offers several features that can be leveraged for this purpose, including layering, version control, and a clear governance framework. We will reason through the first principles to fully address metric drift.

Firstly, we should define what constitutes "metric drift." Metric drift typically occurs when different computations of the same KPI lead to divergent results due to differences in data sources, transformations, or computation logic. The goal is to ensure that all teams computing these metrics are using the exact same definition and computation logic.

#### Triaging the Question

#### Cons
istency Across Dashboards: We need a mechanism to enforce consistent metric definitions.
Centralized Definitions: Ensure that KPIs are defined once, in one place (a central model or seed), then reused across projects.
Version Control and Governance: Track changes to these metrics over time, allowing stakeholders to understand the evolution of KPI calculations.

#### Assumptions
We have multiple dashboards using different dbt models that calculate the same KPIs.
Each dashboard may require its own transformation logic on top of a central definition, but the core metric should remain consistent.
Teams need visibility into the history and changes of these metrics.
Step-by-Step Solution

Centralize Metric Definitions:

Create a metrics schema in your dbt project where you define all KPIs.
Use dbt seed to import reference data or raw definitions from external sources if necessary.

Layered Models for Consistency:

Define the core metric calculation in a base model (e.g., metric_x).

Create a layer that builds upon this base, incorporating any additional transformations needed.

Base Metric Model
Layer 1 Transformation
LAYER 2 TRANSFORMATION
Final Metric

Version Control:

Use dbt docs to generate a documentation site that tracks the evolution of your metrics.
Implement semantic versioning in your model names (e.g., metric_x_v1, metric_x_v2) to capture changes.

Governance and Governance Rules:

Establish a governance policy that all KPIs must be defined centrally before being used by any dashboard or downstream process.
Use dbt packages if necessary, sharing common metric definitions across projects.

Testing and Validation:

Implement unit tests in your dbt models to ensure that the metric logic is correctly applied.
Regularly validate metrics against historical data to catch inconsistencies early.

#### Closing Summary

By centralizing the definition of KPIs using dbt, we can mitigate metric drift by ensuring all calculations are based on a single source of truth. Governance and version control mechanisms provide transparency and traceability, allowing us to manage changes and maintain consistency over time.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to centralize metrics, leading to multiple definitions across different teams.
Mistake 2: Not implementing version control, making it hard to track changes and understand the evolution of KPIs.

#### Follow-Up
How would you handle situations where there are valid reasons for divergent calculations in certain contexts (e.g., time-series adjustments)?

**Comparison:**

| KPI Name | Version | Description | Date |
| --- | --- | --- | --- |
| metric_x_v1 | 1.0 | Core calculation for KPI X | 2023-10-01 |
| metric_x_v1 | 1.1 | Adjusted based on new data sources | 2023-11-01 |

**Code:**

```
MERMAID
Copy
flowchart TD
  A[Base Metric Model] --> B[Layer 1 Transformation]
  C[LAYER 2 TRANSFORMATION] --> D[Final Metric]
```

**Example input / output:**

```text
sequenceDiagram
  participant User
  participant Dashboard A
  participant dbt Project
  participant Metric X Model
  participant Final Calculation

  User->>Dashboard A: Request KPI X
  Dashboard A->>dbt Project: Query metric_x_v1
  dbt Project->>Metric X Model: Compute kpi_value
  Metric X Model->>Final Calculation: Apply additional logic (optional)
  Final Calculation-->>Dashboard A: Return adjusted kpi_value
```

**Data:**

```yaml
# Example of a dbt model with version control
version: 2

models:
  - name: metric_x_v1
    description: "Core calculation for KPI X"
    columns:
      - name: kpi_value
        description: "Computed value based on central definition"

# Layered transformation example
version: 2

models:
  - name: metric_x_v2
    description: "Second version of the KPI, incorporating additional logic"
    depends_on:
      - ref('metric_x_v1')
    columns:
      - name: kpi_value_adjusted
        description: "Adjusted value based on layering transformation"
```

**Flow:**

1. Base Metric Model
2. Layer 1 Transformation
3. LAYER 2 TRANSFORMATION
4. Final Metric

