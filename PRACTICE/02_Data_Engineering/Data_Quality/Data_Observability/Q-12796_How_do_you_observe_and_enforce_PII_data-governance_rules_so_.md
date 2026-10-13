---
type: "practice"
difficulty: "Deep"
---

# How do you observe and enforce PII/data-governance rules so sensitive columns are flagged when they appear unexpectedly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Quality Monitoring](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When implementing data observability for enforcing PII (Personally Identifiable Information) or other data governance rules, the primary goal is to ensure that any unexpected appearance of sensitive columns triggers immediate identification and remediation. This involves a combination of technical tools, policies, and monitoring strategies.

#### Triaging the Question

The core of this question revolves around identifying how to detect and flag sensitive data in real-time while ensuring compliance with organizational data governance rules. The interviewer is probing your understanding of:

Technical Tools: Which systems or frameworks can be used for detecting PII.
Data Governance Policies: How to define and enforce these policies across a diverse dataset.
Monitoring and Alerting: Implementing mechanisms to notify stakeholders when violations occur.

#### Assumptions
The organization has established clear data governance policies regarding which columns contain sensitive information.
There is a need for real-time monitoring of large-scale datasets, possibly in different environments (e.g., production, development).
You have access to metadata describing the schema and relevant PII rules.
Step-by-Step Solution
1. Define Sensitive Columns

First, define which columns contain sensitive information. This could be based on predefined labels or tags within your data catalog.

SQL
CREATE TABLE customer_data (
    id INT,
    name VARCHAR(255),
    email VARCHAR(255) /* Sensitive column */,
    address VARCHAR(255)
);
2. Implement Data Masking and Encryption

Use data masking or encryption techniques to protect sensitive information at rest and in transit.

PYTHON
# Example using Python with a library like pandas for data manipulation
import pandas as pd

def mask_sensitive_columns(df):
    # Mask email column
    df['email'] = df['email'].apply(lambda x: f"{x[:3]}*****{x[-4:]}")
    return df
3. Real-Time Monitoring with Data Observability Tools

Utilize tools like Apache Kafka, AWS Glue, or Google Cloud Dataflow for real-time data processing and monitoring.

Data Source
Apache Kafka
Kafka Streams
Rule Engine
Alert System
4. Define Data Governance Rules

Establish a set of rules for detecting PII in the data, such as regex patterns or specific value checks.

SQL
CREATE OR REPLACE TABLE pii_rule (
    column_name VARCHAR(255),
    rule_expression VARCHAR(255)
);

INSERT INTO pii_rule (column_name, rule_expression) VALUES ('email', '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$');
5. Apply Rules and Monitor Data

Use a rule engine to apply these rules in real-time and generate alerts when violations occur.

PYTHON
import re

def check_email(email):
    return bool(re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email))

# Example check function
def monitor_data(df):
    for index, row in df.iterrows():
        if check_email(row['email']):
            print(f"Alert: Sensitive data detected in row {index}")

# Apply monitoring to a DataFrame
monitor_data(mask_sensitive_columns(pd.read_csv('customer_data.csv')))
6. Alert and Remediation System

Set up an alert system that notifies relevant stakeholders when violations are found.

PYTHON
def send_alert(email, rule_name):
    # Example: Send email or trigger webhook
    print(f"Alert: PII detected in {email}. Rule violated: {rule_name}")
7. Continuous Improvement and Reporting

Regularly review the effectiveness of these mechanisms and update policies as needed.

#### Closing Summary

To effectively observe and enforce PII/data governance rules, you need a combination of technical tools, well-defined policies, and real-time monitoring. Implementing data masking and encryption helps protect sensitive information. Real-time processing frameworks like Apache Kafka can help detect violations in near real-time. A robust rule engine with clear governance policies ensures compliance and timely alerts.

#### Common Mistakes/Follow-Ups
Not Defining Clear Policies: Ensure there are explicit rules for what constitutes PII.
Lack of Real-Time Monitoring: Sensitive data could be flagged after it has already been exposed.
Overlooking Data Ingestion Points: Ensure all sources of data (e.g., APIs, batch jobs) are monitored.

Would you like to explore the topic further or discuss any specific aspect in more detail?

**Comparison:**

```
Column	Type
id	integer
name	varchar
email	varchar
address	varchar
```

**Code:**

```python
# Example code snippets for data masking and real-time monitoring
```

**Example input / output:**

```text
Input: customer_data.csv (example data)
Output:
Alert: Sensitive data detected in row 0. Rule violated: email
```

**Data:**

```json
{
  "customer_data": [
    { "id": 1, "name": "John Doe", "email": "johndoe@example.com", "address": "123 Main St" },
    { "id": 2, "name": "Jane Smith", "email": "janesmith@example.org", "address": "456 Elm St" }
  ]
}
```

**Flow:**

1. Data Source
2. Apache Kafka
3. Kafka Streams
4. Rule Engine
5. Alert System

