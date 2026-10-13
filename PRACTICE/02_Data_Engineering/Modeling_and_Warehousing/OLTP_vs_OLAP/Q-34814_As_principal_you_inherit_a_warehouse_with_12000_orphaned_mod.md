---
type: "practice"
difficulty: "Expert"
---

# As principal you inherit a warehouse with 12000 orphaned models and no clear OLTP-source lineage — how do you drive a rationalization program without breaking downstream dashboards?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Separating OLTP and Reporting](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Separating_OLTP_and_Reporting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When inheriting a warehouse with a large number of orphaned models and no clear OLTP-source lineage, the key challenge is to drive a rationalization program that maintains the integrity of downstream dashboards while minimizing disruption. The solution involves understanding the current state, identifying dependencies, creating a detailed lineage map, and implementing a phased migration strategy.

#### Triage

The question probes your ability to handle complex data warehousing scenarios where you have incomplete or ambiguous information about source systems and their relationships with the warehouse models. You need to demonstrate how to systematically identify and address issues without causing disruptions in critical business operations.

Step-by-Step Solution:

Assessment Phase:

Current State Understanding: Begin by assessing the current state of the data warehouse, focusing on identifying the orphaned models and understanding their structure and usage.
Impact Analysis: Determine the impact of these orphans on downstream dashboards and business processes.

Dependency Mapping:

Lineage Mapping: Develop a detailed lineage map to understand which models are used by various dashboards and reports, and trace back to find their source systems (if available).
Tooling Support: Leverage data management tools like metadata repositories or ETL monitoring solutions that can help visualize the dependencies.

Phased Migration Strategy:

Prioritization: Prioritize models based on their criticality to business operations and usage frequency.
Testing Environment Setup: Set up a testing environment where you can safely experiment with changes without affecting production.
Migration Steps: Implement a migration plan that involves creating new, properly linked models in the data warehouse while maintaining existing ones.

Communication and Collaboration:

Stakeholder Engagement: Engage stakeholders from both the business and IT teams to understand their perspectives and ensure buy-in for any changes.
Documentation: Maintain comprehensive documentation throughout the process to track all changes and dependencies.

Monitoring and Feedback Loop:

Post-Migration Monitoring: Continuously monitor the performance and accuracy of dashboards post-migration.
Feedback Mechanism: Establish a feedback mechanism with end-users to gather real-time insights on any issues encountered.

#### Closing Summary

By systematically assessing, mapping dependencies, implementing a phased migration strategy, engaging stakeholders, and maintaining rigorous monitoring, you can rationalize the warehouse without breaking downstream dashboards. This approach ensures that business operations remain uninterrupted while improving data quality and governance.

#### Common Mistakes/Follow-Ups
Common Mistake: Rushing into changes without thorough understanding of dependencies could lead to unexpected issues.

#### Follow-Up
Question: How would you handle situations where source systems are not readily available for orphaned models?

**Comparison:**

```
Orphaned Models
SalesModel
Revenue Dashboard
Monthly Sales Report
CustomerModel
Customer SegmentationDashboard
```

**Code:**

```sql
-- Example SQL query for lineage mapping
SELECT target_model, source_table, operation_type, timestamp
FROM metadata_repository.lineage_mapping;
```

**Example input / output:**

| Model Name | Status   | Usage (Dashboard/Report)           |
|------------|----------|-----------------------------------|
| SalesModel | Orphaned | Revenue Dashboard - High, Monthly Sales Report - Medium |
| CustomerModel | Active | Customer Segmentation Dashboard - Low  |

**Data:**

```json
{
  "models": [
    {
      "model_name": "SalesModel",
      "status": "Orphaned",
      "usage": [
        { "dashboard_name": "Revenue Dashboard", "impact_level": "High" },
        { "report_name": "Monthly Sales Report", "impact_level": "Medium" }
      ]
    },
    {
      "model_name": "CustomerModel",
      "status": "Active",
      "usage": [
        { "dashboard_name": "Customer Segmentation Dashboard", "impact_level": "Low" }
      ]
    }
  ]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD
3. start((Start))
4. assess((Assessment Phase))
5. map_dependencies((Dependency Mapping))
6. prioritize_models((Prioritization))
7. setup_env((Setup Testing Environment))
8. migrate(models, "Phased Migration Strategy")
9. communicate((Communication and Collaboration))
10. monitor((Monitoring and Feedback Loop))
11. end((End))
12. start --> assess
13. assess --> map_dependencies
14. map_dependencies --> prioritize_models
15. prioritize_models --> setup_env
16. setup_env --> migrate
17. migrate --> communicate
18. communicate --> monitor
19. monitor --> end

