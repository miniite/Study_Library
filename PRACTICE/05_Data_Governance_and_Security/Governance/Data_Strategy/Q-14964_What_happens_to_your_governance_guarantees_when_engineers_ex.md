---
type: "practice"
difficulty: "Deep"
---

# What happens to your governance guarantees when engineers export telemetry into personal notebooks and dashboards for ad-hoc analysis, and how do you architect against that leakage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Platform Telemetry](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Platform_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps environments, ensuring robust data strategy and governance is paramount to maintaining control over sensitive information like telemetry data. When engineers export this data into personal notebooks or dashboards for ad-hoc analysis, it can lead to breaches in governance guarantees, such as data privacy, security, compliance, and consistency. To architect against such leakage, we need a layered approach combining technical controls and organizational policies.

The question is probing how you manage the risks associated with ad-hoc data usage outside of your established governed systems. We must consider assumptions like the scale of exportation, types of data involved (e.g., PII, confidential telemetry), frequency of such exports, and the potential impact on overall data governance frameworks.

Let's break down the problem from first principles:

Data Classification: First, classify the telemetry data into different categories based on sensitivity (e.g., public, private, confidential). This helps in defining access controls and audit policies.

Governance Frameworks: Establish a comprehensive data governance framework that includes data stewards, data owners, and clear policies for how data should be handled.

Data Export Controls: Implement technical controls to prevent unauthorized exportation of sensitive data. For example, enforce usage of approved tools and platforms for exporting telemetry data.

Data Masking & Encryption: Apply data masking and encryption techniques to protect PII and other sensitive information when it is exported or stored in personal notebooks or dashboards.

Access Controls & Auditing: Implement robust access controls and continuous auditing mechanisms to monitor who has accessed which data and how they are using it.

Data Warehouse vs Notebooks/Dashboards: Use a centralized, governed data warehouse for ad-hoc analysis whenever possible. This ensures that all data queries and analyses adhere to established governance policies.

Compliance & Legal Considerations: Ensure compliance with relevant regulations (e.g., GDPR, CCPA) by incorporating legal requirements into your governance strategy.

Training & Awareness Programs: Regularly train engineers on the importance of adhering to data governance policies and the potential consequences of non-compliance.

Now let's delve deeper into each aspect:

Detailed Solution

Data Classification

Classify telemetry data into categories such as Public (open to everyone), Private (restricted access), and Confidential (highly sensitive).

#### Example
"All user login information is classified as PII and should be treated with strict confidentiality."

Governance Frameworks

Define roles and responsibilities for data stewards, owners, and users.
Create policies on how to handle sensitive data, such as access levels and usage scenarios.

Data Export Controls

Enforce the use of specific tools or platforms for exporting telemetry data.

#### Example
"All exports must be conducted through a secure portal that logs all actions."

Data Masking & Encryption

Apply masking techniques to PII fields when stored in personal notebooks/dashboards.
Use encryption at rest and in transit to protect sensitive data.

Access Controls & Auditing

Implement role-based access controls (RBAC) and least privilege principles.
Regularly audit access logs to detect any unauthorized activities.

Data Warehouse vs Notebooks/Dashboards

Use a governed data warehouse for ad-hoc analysis, ensuring that all queries are audited and controlled.

#### Example
"All ad-hoc analyses should be run in the secure data warehouse, not directly from personal notebooks."

Compliance & Legal Considerations

Integrate compliance requirements into your governance framework to ensure ongoing adherence to legal standards.

Training & Awareness Programs

Conduct regular training sessions on data governance policies and best practices.
Provide clear documentation and reminders about the importance of following these guidelines.

#### Closing Summary

By implementing a layered approach that combines technical controls with strong organizational policies, we can mitigate the risks associated with engineers exporting telemetry data into personal notebooks or dashboards. Key steps include data classification, establishing robust governance frameworks, enforcing access controls, using data masking and encryption, and providing continuous training to maintain awareness.

#### Common Mistakes
& Follow-Ups
Ignoring Data Classification: Failing to classify data can lead to inadequate security measures.
Lack of Access Controls: Weak access controls allow unauthorized users to access sensitive information.
Not Using Governed Tools: Reliance on personal tools for ad-hoc analysis bypasses governance, leading to potential data leaks.

#### Follow-ups
might include discussing specific tools and platforms used for governed data access or exploring how machine learning can be employed to detect and prevent unauthorized data exports.

**Comparison:**

```
Data Type	Sensitive Information	Access Level
Public	Log Types	Open
Private	User Activity Logs	Restricted
Confidential	Sensitive Metrics	Highly Restricted
```

**Code:**

```python
# Example Python code snippet for implementing a simple data export control system

def export_data(data, export_type):
    if not is_authorized_user():
        raise PermissionError("Unauthorized user detected")

    if export_type == 'notebook':
        mask_pii_fields(data)
        encrypt_data(data)

    # Additional checks and controls can be added here
```

**Example input / output:**

```

Input:

Data: Telemetry data including PII (user IDs, login times)
Export Type: Notebook

Output:

Masking PII Fields: User IDs and login times are masked.
Encryption: Data is encrypted before export.
JSON
Copy
{
  "masked_data": {
    "user_ids": ["[MASKED]"],
    "login_times": ["[MASKED]"]
  },
  "encrypted_data": true
}
```

**Data:**

```json
{
  "data_classification": {
    "public": ["log_types: system"],
    "private": ["log_types: user_activity", "log_types: session_details"],
    "confidential": ["log_types: sensitive_metrics"]
  }
}
```

**Flow:**

1. No
2. Yes
3. Data Export Request
4. Is Authorized User?
5. Deny Access
6. Mask PII Fields
7. Encrypt Data
8. Export Data

