---
type: "practice"
difficulty: "Deep"
---

# How do you reconcile MDM survivorship with 'source of truth by attribute' governance — some attributes are authoritative in CRM, others in ERP — how is that policy encoded and enforced?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-domain MDM Programs](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Multi-domain_MDM_Programs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In Master Data Management (MDM), survivorship policies determine which data sources provide authoritative values for specific attributes across different systems like CRM and ERP. The 'source of truth by attribute' governance ensures that each system is responsible for certain attributes, but reconciling this with survivorship requires a robust architecture to ensure consistency and correctness in MDM. This involves defining clear rules, implementing them via data pipelines, and continuously monitoring the outcomes.

#### Triaging the Question

The core question here is about how to integrate 'source of truth by attribute' governance into MDM systems while handling survivorship policies effectively. The interviewer is probing your understanding of:

Attribute Ownership: Understanding which attributes are authoritative in CRM versus ERP.
Survivorship Policies: How data from different sources are reconciled when there's a conflict.
Governance Rules: Defining and enforcing these rules across multiple systems.
Data Consistency: Ensuring that MDM maintains the correct state of truth based on the authoritative source.

#### Assumptions
We have CRM and ERP as primary data sources with distinct attribute ownership.
The MDM system must maintain a consistent view while respecting the 'source of truth by attribute' governance.
There might be conflicts where both systems hold different values for an attribute, requiring a reconciliation strategy.
Step-by-Step Solution from First Principles

Attribute Ownership Definition:

Define which attributes are authoritative in CRM and ERP.
This can be done through metadata definitions that specify the source of truth per attribute.

Survivorship Policy Design:

Develop policies to handle conflicts when both systems have different values for an attribute.
Common strategies include:
Choosing the most recent update.
Using a heuristic based on historical data freshness.
Applying business rules to prioritize certain attributes over others.

Data Pipeline Architecture:

Design a data pipeline that integrates data from CRM and ERP, applying survivorship policies as data is ingested into MDM.
Use ETL (Extract, Transform, Load) processes with transformation logic built around the defined rules.

Governance Enforcement:

Implement governance controls through metadata management tools or custom scripts.
Regularly audit MDM data to ensure compliance with the 'source of truth by attribute' policy.

Monitoring and Validation:

Set up monitoring systems to track data quality and identify potential conflicts.
Use alerts and dashboards to notify stakeholders about any deviations from expected behavior.
Detailed Example

Let's say we have an MDM system serving customer data with CRM holding primary contact information and ERP managing financial transactions. The attributes are:

Contact_Name (CRM authoritative)
Account_Responsibility (ERP authoritative)

A sample workflow could be:

Data Ingestion:

Extract data from CRM and ERP.
Transform the data using a pipeline to apply survivorship rules.

Survivorship Rule Application:

If both systems have different values for Contact_Name, choose the most recent update based on timestamp.
For Account_Responsibility, ensure that only ERP updates are applied since it's its authoritative source.

Data Validation:

Post-ingestion, validate data against predefined rules to ensure consistency and correctness.
Use a validation engine that cross-checks the MDM records with the sources.

#### Closing Summary

In conclusion, reconciling 'source of truth by attribute' governance with survivorship policies involves defining clear attribute ownership, designing robust survivorship rules, implementing them through well-architected data pipelines, and continuously validating and monitoring the MDM system. This ensures that each system's authoritative data is respected while maintaining a consistent overall view in MDM.

#### Common Mistakes/Follow-Ups
Overly Complex Rules: Over-engineering the rules can lead to complexity and potential errors.
Lack of Monitoring: Failing to set up robust monitoring systems can result in undetected data inconsistencies.
Ignoring Historical Data: Not considering historical context might cause incorrect choices during survivorship.

Would you like me to elaborate on any specific aspect or provide a more detailed example?

**Comparison:**

| Customer ID | Contact Name (CRM) | Account Responsibility (ERP) |
|-------------|--------------------|------------------------------|
| 1           | Alice              | Sales                        |
| 2           | Bob                | Finance                      |

**Code:**

```python
# Example Python code for applying survivorship rules using Pandas

import pandas as pd

def apply_survivorship_rules(crm_data, erp_data):
    # Merge data frames based on common key (e.g., customer ID)
    merged_data = crm_data.merge(erp_data, on='customer_id', suffixes=('_crm', '_erp'))

    # Apply survivorship rules
    for index, row in merged_data.iterrows():
        if row['contact_name_crm'] != row['contact_name_erp']:
            merged_data.at[index, 'contact_name'] = max(row['timestamp_crm'], row['timestamp_erp'])
        else:
            merged_data.at[index, 'contact_name'] = row['contact_name_crm']

    return merged_data
```

**Example input / output:**

```text
Input:
CRM Data:
[{"customer_id": 1, "contact_name": "Alice", "timestamp": "2023-01-01T10:00:00Z"},
 {"customer_id": 2, "contact_name": "Bob", "timestamp": "2023-02-01T11:00:00Z"}]

ERP Data:
[{"customer_id": 1, "account_responsibility": "Sales", "timestamp": "2023-01-01T15:00:00Z"},
 {"customer_id": 2, "account_responsibility": "Finance", "timestamp": "2023-02-02T16:00:00Z"}]

Output:
Merged and Survived Data:
[{"customer_id": 1, "contact_name": "Alice", "account_responsibility": "Sales"},
 {"customer_id": 2, "contact_name": "Bob", "account_responsibility": "Finance"}]
```

**Data:**

```json
{
  "crm_data": [
    {"customer_id": 1, "contact_name": "Alice", "timestamp": "2023-01-01T10:00:00Z"},
    {"customer_id": 2, "contact_name": "Bob", "timestamp": "2023-02-01T11:00:00Z"}
  ],
  "erp_data": [
    {"customer_id": 1, "account_responsibility": "Sales", "timestamp": "2023-01-01T15:00:00Z"},
    {"customer_id": 2, "account_responsibility": "Finance", "timestamp": "2023-02-02T16:00:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Ingestion
6. Is Attribute Authoritative inCRM?
7. Merge with CRM Data
8. Is Attribute Authoritative inERP?
9. Merge with ERP Data
10. Apply Heuristic Rules
11. Merge and Apply SurvivorshipRules
12. Validate MDM Records

