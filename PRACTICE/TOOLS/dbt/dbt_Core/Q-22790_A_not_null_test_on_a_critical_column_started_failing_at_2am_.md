---
type: "practice"
difficulty: "Intermediate"
---

# A not_null test on a critical column started failing at 2am. How do you triage severity, decide whether to block downstream, and communicate to consumers?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ dbt Tests](<../../../../TOOLS/dbt/dbt_Core/dbt_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a not_null test on a critical column in dbt starts failing at 2am, it's crucial to quickly assess the situation, decide on an appropriate response, and communicate effectively with stakeholders. This triage involves understanding the impact, assessing the risk, deciding whether to block downstream processes, and ensuring clear communication.

#### Triage
Understanding the Problem
Identify the Failed Test: The not_null test failing means that a value in the critical column is null or empty.
Criticality of the Column: Determine which table and column this is affecting. Critical columns often contain essential data for downstream processes, reporting, or business decisions.
Time of Failure: The fact that it happened at 2am (likely not during typical operational hours) suggests an issue outside regular monitoring windows.

#### Assumptions
You have a dbt project with clear naming conventions and metadata.
The dbt model has a not_null constraint on the critical column.
There are appropriate alerting mechanisms in place, such as Slack notifications or email alerts for test failures.
Step-by-Step Solution
Re-run the Test Locally:
Use dbt test --select <model_name> to re-execute the specific model and confirm if the failure is reproducible.
Check Recent Changes:
Review recent commits in your Git history (git log) to see if any changes might have impacted this column.
Inspect Data Source:
Check the source system or API that feeds data into dbt to ensure it’s providing valid data.
Validate Schema Changes:
Ensure no schema changes inadvertently removed not_null constraints.
Review ETL Processes:
Inspect any ETL pipelines (e.g., Apache Airflow, Prefect) feeding this table for potential issues.
Check Data Quality Jobs:
Look at other data quality jobs or workflows that may have been impacted.
Decision-Making
Severity Assessment: Evaluate the criticality of the data and its impact on downstream processes. For instance, a null value in an ID column would be more severe than one in a descriptive field.
Block Downstream Processes: If the failure impacts key reports or business-critical workflows, consider blocking them to prevent using potentially incorrect data.
Communication Strategy
Immediate Alert: Notify stakeholders via your alerting mechanism (e.g., Slack) with a brief message indicating the failure and next steps.
Detailed Report: Prepare a more detailed report for review by key stakeholders:
Title: Critical dbt Model Failure - [Model Name]
Summary:
CODE
At 2am, a 'not_null' test on critical column [column_name] in model [model_name] failed. The issue appears to be due to null values in this column.
Impact Assessment: Highlight the potential impact of this failure (e.g., invalid reports, incorrect business decisions).
Next Steps:
Investigate further by re-running tests and checking data sources.
Potential blockage of downstream processes until resolved.

#### Common Mistakes/Follow-Ups
Ignoring the Alert: Failing to acknowledge or respond to alerts can lead to compounded issues.
Over-Blocking: Blocking all downstream processes without a clear understanding of their interdependencies may cause unnecessary downtime.
Lack of Detail in Reporting: Providing insufficient context can delay resolution and impact user trust.

#### Closing Summary

In summary, the key steps are re-running tests, identifying recent changes, checking data sources, validating schema integrity, and reviewing ETL processes. Deciding whether to block downstream processes depends on the criticality of the affected data. Effective communication is crucial throughout this process to ensure stakeholders are informed and can take appropriate actions.

**Comparison:**

```
tableDiagram
    dbt_model -->|Re-run| local_test_results
    local_test_results -->|Identify changes| recent_commits
    local_test_results -->|Check source| data_source_check
    local_test_results -->|Validate schema| schema_validation
    local_test_results -->|Review ETL| etl_review
```

**Code:**

```sql
-- Example SQL query to re-run a specific model test
dbt test --select my_model
```

**Example input / output:**

| Input                | Output                    |
|----------------------|---------------------------|
| `dbt test --select sales_fact` | Test run results, indicating failure in 'not_null' constraint on 'customer_id' column. |

**Data:**

```json
{
  "model_name": "sales_fact",
  "column_name": "customer_id"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Re-run Test
10. Is Failure Reproducible?
11. Identify Recent Changes
12. Check Data Source
13. Is Change Related to Issue?
14. Validate Schema
15. Review ETL Process
16. Is Issue Data Ingestion?
17. Investigate ETL Issues
18. Notify Stakeholders
19. Resolve and Test
20. Block Downstream Processes?
21. Implement Blockage
22. Continue Investigation

