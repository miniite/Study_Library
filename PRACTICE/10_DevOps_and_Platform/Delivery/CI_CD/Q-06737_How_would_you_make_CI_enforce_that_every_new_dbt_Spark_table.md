---
type: "practice"
difficulty: "Deep"
---

# How would you make CI enforce that every new dbt/Spark table has an owner, a description, freshness monitoring, and a data-classification tag before it can be deployed?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data and Analytics CI Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that every new dbt or Spark table has an owner, a description, freshness monitoring, and a data-classification tag before deployment, we need to integrate CI/CD practices with our Data Engineering pipeline. This involves setting up automated checks within our Continuous Integration (CI) environment to validate these attributes.

#### Triaging the Question

The core of this question revolves around implementing robust validation mechanisms in the CI process to enforce best practices for data management and governance. We are probing your understanding of:

Data Governance Principles: Ensuring that every dataset is properly tagged and described.
CI/CD Pipeline Integration: How these checks can be integrated into an existing or new CI pipeline.
Tooling and Automation: The tools and methods required to automate the validation process.

#### First-Principles Solution

To enforce this, we will:

Define Validation Rules: Create a set of rules that check for the presence of necessary attributes (owner, description, freshness monitoring, data-classification tag) in each new table.
Integrate with CI/CD Pipeline: Use our CI/CD tooling to run these validation checks automatically during the build phase.
Automate and Scale: Ensure the solution is scalable and can handle multiple datasets efficiently.
Step-by-Step Solution

Define Validation Rules:

Owner: Every table should have an owner specified, typically a user or role within our organization.
Description: A brief description of what the data represents is required for context.
Freshness Monitoring: This involves setting up alerts and checks to ensure that the data is current.
Data-Classification Tag: Tags such as "Sensitive," "Public," etc., should be applied based on the nature of the data.

Integrate with CI/CD Pipeline:

Choose a CI tool like Jenkins, GitLab CI, or GitHub Actions to automate these checks.
Use dbt or Spark metadata APIs to retrieve and validate table information.
Implement custom scripts or use existing plugins in our CI tooling to run the validation logic.

Automate and Scale:

Ensure that the validation runs are idempotent, meaning they can be rerun without producing different results.
Leverage parallel execution capabilities of CI/CD tools to speed up the process when multiple tables need validation.

#### Example
Implementation
Yes
No
Yes
No
Yes
No
Yes
No
Code Commit
Run Build
Is Owner Set?
Proceed with Build
Fail Build, Notify Developer
Is Description Set?
Proceed with Build
Fail Build, Notify Developer
Is Freshness MonitoringConfigured?
Proceed with Build
Fail Build, Notify Developer
Is Data-Classification Tag Set?
Proceed with Build
Fail Build, Notify Developer

#### Closing Summary

By integrating these validation checks into our CI/CD pipeline, we can ensure that all new dbt or Spark tables meet best practices before deployment. This not only improves data quality and governance but also reduces the risk of issues arising from poorly defined or unmonitored datasets.

#### Common Mistakes
and Follow-Ups
Not Handling Edge Cases: Ensure that edge cases like missing metadata are handled gracefully.
Overlooking Automated Notifications: Implement mechanisms to notify developers if their table fails validation, helping them address the issues promptly.
Lack of Scalability: Make sure the solution scales well with increasing numbers of tables and complex datasets.

By addressing these points, we can build a robust CI/CD pipeline that enforces data governance principles efficiently.

**Comparison:**

```
Table Name	Owner	Description	Freshness Monitoring	Data Classification
sales_data_q1_2023	data_engineer	Sales data for Q1 2023	true	Non-sensitive
```

**Code:**

```python
import dbt_metadata

def validate_table(table_name):
    metadata = dbt_metadata.get_table_metadata(table_name)

    if not metadata['owner']:
        raise ValueError(f"Owner is missing for {table_name}")

    if not metadata['description']:
        raise ValueError(f"Description is missing for {table_name}")

    freshness_monitoring_status = check_freshness_monitoring(metadata)
    if not freshness_monitoring_status:
        raise ValueError(f"Freshness monitoring is not configured for {table_name}")

    data_classification_tag = get_data_classification_tag(metadata)
    if not data_classification_tag:
        raise ValueError(f"Data classification tag is missing for {table_name}")

def check_freshness_monitoring(metadata):
    # Check freshness monitoring implementation
    pass

def get_data_classification_tag(metadata):
    # Retrieve and validate data classification tag
    pass
```

**Example input / output:**

```text
Input:
- A new dbt table 'sales_data_q1_2023'

Output:
- Validation Passes: Owner, Description, Freshness Monitoring, Data-Classification Tag are all present and valid.
- Notification to Developer if any validation fails.
```

**Data:**

```json
{
  "metadata": {
    "owner": "data_engineer",
    "description": "Sales data for Q1 2023",
    "freshness_monitoring": true,
    "data_classification_tag": "Non-sensitive"
  }
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
9. Code Commit
10. Run Build
11. Is Owner Set?
12. Proceed with Build
13. Fail Build, Notify Developer
14. Is Description Set?
15. Proceed with Build
16. Fail Build, Notify Developer
17. Is Freshness MonitoringConfigured?
18. Proceed with Build
19. Fail Build, Notify Developer
20. Is Data-Classification Tag Set?
21. Proceed with Build
22. Fail Build, Notify Developer

