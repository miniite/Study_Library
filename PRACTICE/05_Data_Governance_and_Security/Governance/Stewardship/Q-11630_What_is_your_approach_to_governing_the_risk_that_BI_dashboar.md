---
type: "practice"
difficulty: "Deep"
---

# What is your approach to governing the risk that BI dashboards silently diverge because each analyst re-implements the same metric with slightly different SQL?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Preventing PII Leakage](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Preventing_PII_Leakage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern the risk of BI dashboards diverging due to analysts independently reimplementing metrics with slight variations, a robust data governance and stewardship strategy is essential. This involves creating standardized definitions, ensuring consistency in metric calculations, and maintaining transparent lineage tracking. At its core, this approach aims to centralize control over how metrics are calculated, reducing the risk of discrepancies.

#### Triaging the Question

The question is asking about strategies to ensure that BI dashboards remain consistent across different analysts. Specifically, it probes our ability to manage and standardize metric definitions, ensuring that variations in SQL implementations do not lead to divergent results. Key assumptions include:

The existence of multiple analysts using diverse data sources.
Each analyst potentially has their own interpretation or implementation of metrics.
The need for transparency and traceability in how metrics are calculated.

#### First Principles Solution

To address this issue, we must follow a multi-faceted approach:

Standardize Metric Definitions: Define clear, unambiguous metric definitions that all analysts must adhere to. This includes documenting the business logic behind each metric.
Centralized Calculation Engine: Develop or utilize an internal or external system for calculating metrics where all queries are run from a single source of truth.
Version Control and Lineage Tracking: Implement mechanisms to track changes in metric definitions and calculations, ensuring that any modifications can be audited and understood.
Training and Documentation: Provide comprehensive training and documentation for analysts on how to correctly implement standardized metrics.
Detailed Steps

Define Metric Specifications: Document each metric with a clear definition, including the business context, the source data used, and the calculation logic. Use standard language or templates to ensure consistency.

#### Example
"Monthly Active Users (MAU) is defined as users who have logged in at least once during the month."

Implement Centralized Calculation: Develop a centralized system where all metric calculations are performed. This could be part of your data warehouse, an API layer, or a dedicated analytics engine.

#### Example
A SQL function calculate_MAU that takes input from source tables and returns the calculated MAU.

Use Version Control for Metrics: Store metric definitions in version-controlled repositories to track changes over time. Ensure that all analysts reference these versions when implementing metrics.

#### Example
Git repository with branches representing different versions of metric definitions.

Audit and Review Processes: Establish regular review processes where metrics are audited by a governance team to ensure compliance with standard definitions.

#### Example
Monthly audits conducted by a data governance committee.

Training Programs: Provide ongoing training for analysts on best practices, common pitfalls, and the importance of adhering to standardized definitions.

#### Example
Regular workshops and refresher sessions.

#### Closing Summary

By implementing these strategies, we can significantly reduce the risk of BI dashboards diverging due to inconsistent metric implementations. Centralizing control over metric calculations ensures that all results are aligned with predefined standards, enhancing the reliability and consistency of analytics across the organization.

#### Common Mistakes/Follow-Ups

Failing to document and standardize metrics properly.
Not implementing a centralized calculation engine, leading to duplicated effort.
Insufficient training and support for analysts, resulting in inconsistent implementation practices.
Lack of regular audits and reviews, allowing deviations from standards to persist unnoticed.

**Comparison:**

| Version | Changes                    |
|---------|----------------------------|
| 1       | Added support for mobile logins |
| 2       | Updated logic to exclude guest users |

**Code:**

```sql
-- Example SQL function for calculating MAU
CREATE OR REPLACE FUNCTION calculate_MAU(start_date DATE, end_date DATE)
RETURNS TABLE(user_id INT, login_count INT) AS $$
BEGIN
    RETURN QUERY
    SELECT user_id, COUNT(*) as login_count
    FROM logins
    WHERE login_date BETWEEN start_date AND end_date
    GROUP BY user_id;
END; $$ LANGUAGE plpgsql;
```

**Example input / output:**

```text
Input:
- start_date = '2023-09-01'
- end_date = '2023-09-30'

Output (using the centralized calculation engine):
{
  "user_id": [100, 101, 102],
  "login_count": [45, 60, 30]
}
```

**Data:**

```json
{
  "metric_definitions": {
    "monthly_active_users": {
      "description": "Users who have logged in at least once during the month",
      "source_tables": ["logins"],
      "calculation_logic": "count distinct user_id where login_date between start_date and end_date"
    }
  },
  "versions_history": [
    {"version": 1, "timestamp": "2023-09-15", "changes": ["Added support for mobile logins"]},
    {"version": 2, "timestamp": "2023-10-01", "changes": ["Updated logic to exclude guest users"]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is metric defined?
5. Centralized Calculation Engine
6. Define Metric and SourceTables
7. Audit and Review Process
8. Document Metric Definition

