---
type: "practice"
difficulty: "Expert"
---

# How do you handle governance for a merger where two companies have overlapping customer datasets with conflicting definitions, keys, and quality?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Canonical Model vs Domain Autonomy](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Canonical_Model_vs_Domain_Autonomy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Handling data governance in a merger is a complex task that requires careful planning and execution to ensure consistency, accuracy, and compliance across overlapping datasets. When two companies have datasets with conflicting definitions, keys, and varying levels of quality, it's crucial to establish a comprehensive approach that prioritizes both short-term integration and long-term governance.

#### Triage
Understanding the Problem

The question is essentially asking how we can manage data governance during a merger where there are inconsistencies in customer datasets. The interviewer is probing our ability to:

Identify the key challenges such as conflicting definitions, keys, and quality issues.
Develop robust strategies for resolving these conflicts.
Ensure that the integration process aligns with organizational goals, legal requirements, and industry standards.
First Principles: Establishing a Governance Framework

To address this challenge, we need to start by understanding the current state of both datasets and then build a governance framework that can integrate them effectively.

Step 1: Assess Current State
Data Quality: Evaluate the quality of data in both datasets. This includes checking for missing values, duplicates, and outliers.
Key Definitions: Understand the definitions of key fields such as customer identifiers, addresses, etc., to identify inconsistencies.
Customer Overlaps: Identify overlaps in customers between the two datasets.
Step 2: Define Common Standards
Unified Data Model: Develop a unified data model that can accommodate both sets of data. This involves defining standardized field names and datatypes.
Data Quality Rules: Establish rules to ensure data quality, such as mandatory fields, validation checks, etc.
Step 3: Conflict Resolution Strategy
Conflict Detection Mechanism: Implement a mechanism to detect conflicts between the two datasets. This could be based on comparing customer IDs or other unique identifiers.
Manual Review and Approval Processes: For critical conflicts, implement manual review processes where domain experts can approve which version of the data is correct.
Automated Resolution Rules: Develop automated rules for resolving simple conflicts, such as choosing the more recent record.
Step 4: Data Integration
Incremental Migration Strategy: Use an incremental migration strategy to integrate new customer records from one dataset into the other. This ensures that existing customer data remains intact while new or updated records are added.
Batch Processing: Implement batch processing jobs to handle large volumes of data efficiently.
Step 5: Continuous Monitoring and Maintenance
Data Quality Metrics: Continuously monitor data quality metrics such as completeness, accuracy, and consistency.
Governance Policies: Document governance policies and ensure they are followed by all stakeholders.
Training and Awareness: Provide training to all relevant personnel on the new data governance processes.

#### Closing Summary
Comprehensive Governance Strategy

A comprehensive data governance strategy for a merger should focus on understanding the current state of both datasets, establishing common standards, resolving conflicts through a mix of automated and manual processes, integrating the data incrementally, and continuously monitoring and maintaining data quality. The key is to ensure that all steps are documented, reviewed by domain experts, and followed consistently.

#### Common Mistakes/Follow-Ups
Lack of Data Quality Assessment: Not adequately evaluating the quality of both datasets can lead to significant issues down the line.
Overreliance on Automated Solutions: Relying solely on automated conflict resolution mechanisms without human oversight can result in incorrect decisions being made.
Insufficient Documentation and Training: Failing to document governance policies and provide adequate training can lead to inconsistencies and misuse of data.

**Comparison:**

| Customer Overlap Analysis |
|---------------------------|
| Company A ID | Company B ID | Name       |
|--------------|--------------|------------|
| 101          | 101          | Alice      |
| 103          | -            | Charlie    |

**Code:**

```python
# Example of a simple conflict detection mechanism using pandas
import pandas as pd

def detect_conflicts(left_df, right_df, key_columns):
    merged = left_df.merge(right_df, on=key_columns, how='outer', suffixes=('_left', '_right'))
    conflicts = merged[(merged.apply(lambda row: row.left_isnull() & row.right_notnull(), axis=1)) |
                       (merged.apply(lambda row: row.left_notnull() & row.right_isnull(), axis=1))]
    return conflicts
```

**Example input / output:**

```text
Input Data:
Company A Customer Dataset:
| customer_id | name      | email               |
|-------------|-----------|---------------------|
| 101         | Alice     | alice@example.com   |
| 102         | Bob       | null                |
| 103         | Charlie   | charlie@example.com |

Company B Customer Dataset:
| customer_id | name      | email               |
|-------------|-----------|---------------------|
| 101         | Alice     | alice@example.com   |
| 104         | David     | david@example.com   |

Output Data after Integration (Unified Model):
| customer_id | name       | email               |
|-------------|------------|---------------------|
| 101         | Alice      | alice@example.com   |
| 102         | Bob        | null                |
| 103         | Charlie    | charlie@example.com |
| 104         | David      | david@example.com   |
```

**Data:**

```json
{
  "customer_ids": [
    {"id": 101, "name": "Alice", "email": "alice@example.com"},
    {"id": 102, "name": "Bob", "email": null},
    {"id": 103, "name": "Charlie", "email": "charlie@example.com"}
  ]
}
```

**Flow:**

1. Data Quality Assessment
2. Identify Conflicts
3. Unified Data Model
4. Conflict Resolution Strategy
5. Data Integration
6. Continuous Monitoring andMaintenance

