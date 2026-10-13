---
type: "practice"
difficulty: "Intermediate"
---

# What's the role of tags versus classifications versus policies in a governance tool? Give a concrete example chaining all three.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Glossary Metadata and Contracts](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Glossary_Metadata_and_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data governance, "tags," "classifications," and "policies" play distinct but interconnected roles that help ensure data quality, security, and compliance. Tags are metadata labels attached to datasets or assets, classifications categorize data based on its sensitivity or purpose, and policies define the rules and procedures for managing data according to these categories.

Let's dive into a concrete example of how tags, classifications, and policies interlink in a governance tool:

Imagine you're working with a healthcare organization that handles sensitive patient information. Your goal is to implement robust data governance practices to manage this data effectively.

Tags: Metadata Labels

Tags are like sticky notes that help describe the dataset's attributes or characteristics. For example, you might tag datasets with identifiers such as "patient ID," "clinical record," or "diagnostic imaging." These tags provide quick insights into what the data contains but don't necessarily convey its significance in terms of security and compliance.

Classifications: Data Categorization

Classifications are more about categorizing data based on its sensitivity. In our healthcare context, you might classify patient records as "highly sensitive" or "moderately sensitive." This classification helps determine the appropriate level of protection needed for each dataset. For instance, highly sensitive data could be encrypted and stored in a secure environment, while moderately sensitive data might require less stringent measures.

Policies: Rules and Procedures

Policies are the rules that govern how data should be handled based on its classifications. For example, you might have a policy stating that all "highly sensitive" patient records must be reviewed annually by a compliance officer to ensure they comply with HIPAA regulations. Another policy might dictate that any changes to highly sensitive data require dual approval.

#### Example Scenario

#### Cons
ider the following scenario:

Tagging: You tag a dataset as "patient records."
Classifying: You classify this dataset as "highly sensitive" because it contains personal health information.
Policies: Based on the classification, you implement policies that require:
The data to be encrypted both at rest and in transit.
Access controls ensuring only authorized personnel can view or modify the data.
Annual security audits of this dataset by a compliance officer.

#### Triaging the Question

The question is asking for an understanding of how these three elements (tags, classifications, policies) work together to manage data governance effectively. The key here is to show how each element contributes uniquely and interdependently.

Tags help identify datasets.
Classifications define the sensitivity levels.
Policies outline the specific actions required for handling data based on its classification.

#### First-Principles Solution
Tagging: Attach metadata tags to datasets, such as "patient records," "financial transactions," or "confidential documents."
Classifying: Categorize these datasets into predefined sensitivity levels like "highly sensitive," "moderately sensitive," and "low risk."
Policies: Define rules for each classification level. For instance:
Highly Sensitive: Encryption, strict access controls, annual security audits.
Moderately Sensitive: Data masking, regular backups, semi-annual reviews.
Low Risk: Basic logging, periodic scans.

#### Closing Summary

In summary, tags help identify and describe datasets; classifications categorize data based on its sensitivity; and policies define the rules for handling data according to these categories. By chaining these elements together, you create a robust data governance framework that ensures data quality, security, and compliance.

#### Common Mistakes/Follow-Ups
Failing to distinguish between tags and classifications: Tags provide metadata, while classifications define sensitivity levels.
Overlooking the importance of policies: Policies ensure that the correct actions are taken based on classification levels.
Ignoring edge cases: For example, handling data breaches or changes in regulatory requirements.

**Comparison:**

```
Data Set	Tags	Classification	Policies
Patient Records	patient_id, clinical_data	highly_sensitive	Encrypt_both_ends, access_control
Financial Records	account_number, transaction_amount	moderately_sensitive	data_masking, backup
```

**Code:**

```python
# Example Python code for tagging a dataset
tags = {
    "patient_records": ["patient_id", "clinical_data"],
    "financial_transactions": ["account_number", "transaction_amount"]
}

classification = {
    "highly_sensitive": ["HIPAA", "GDPR"],
    "moderately_sensitive": ["compliance", "audit_required"]
}

policies = {
    "highly_sensitive": {"rules": ["encrypt_both_ends", "access_control"], "frequency": "annual"},
    "moderately_sensitive": {"rules": ["data_masking", "backup"], "frequency": "semi-annual"}
}
```

**Example input / output:**

```text
Input:
Dataset Type: patient_records
Classification: highly_sensitive

Output:
Encryption Enabled: True
Access Controls Applied: True
Annual Security Audit Scheduled: Yes
```

**Data:**

```json
{
  "tags": {
    "patient_records": ["patient_id", "clinical_data"],
    "financial_transactions": ["account_number", "transaction_amount"]
  },
  "classifications": {
    "highly_sensitive": ["HIPAA", "GDPR"],
    "moderately_sensitive": ["compliance", "audit_required"]
  },
  "policies": {
    "highly_sensitive": {"rules": ["encrypt_both_ends", "access_control"], "frequency": "annual"},
    "moderately_sensitive": {"rules": ["data_masking", "backup"], "frequency": "semi-annual"}
  }
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Dataset
5. Is it tagged?
6. Tagged
7. Classify
8. Is the dataset highly sensitive?
9. Policies for Highly SensitiveData
10. Policies for ModeratelySensitive Data

