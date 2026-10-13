---
type: "practice"
difficulty: "Expert"
---

# How do you enforce that PII classification and masking policies are applied uniformly at product publish time across all domains?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Ownership vs Cross-cutting Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Domain_Ownership_vs_Cross-cutting_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In a Data Mesh architecture, ensuring uniform application of Personally Identifiable Information (PII) classification and masking policies is crucial for maintaining data governance and privacy compliance. This involves defining clear rules, implementing robust enforcement mechanisms, and continuously monitoring to ensure consistency across all domains.

#### Triaging the Question

The core of this question revolves around enforcing policy uniformity in a Data Mesh environment. The interviewer is probing into your understanding of how policies are defined, enforced, and monitored at publish time. Key assumptions include:

A mature governance framework is in place.
Policies are well-defined and consistent across domains.
Automated tools support enforcement.

#### First-Principles Solution
Policy Definition: Define PII classification rules that cover all possible identifiers (e.g., name, social security number, email).
Metadata Management: Use a metadata management system to capture and standardize these rules across the mesh.
Automated Policy Enforcement: Implement tools that automatically apply masking and classify data based on predefined policies at publish time.
Monitoring & Auditing: Continuously monitor and audit compliance using logs, alerts, and reports.
Step-by-Step Solution

Define PII Classification Rules

Identify common types of PII (e.g., name, address, SSN).
Create a comprehensive list of regex patterns or data templates for these identifiers.
Store this metadata in a central repository accessible to all domains.

Metadata Management System

Use a centralized metadata management system like AWS Glue, Azure Data Catalog, or an open-source solution like Apache Atlas.
Ensure that metadata includes PII classification rules and their enforcement details (e.g., masking levels, encryption methods).

Automated Policy Enforcement

Implement data processing pipelines using frameworks like Apache Flink, Spark Streaming, or AWS Glue ETL jobs.
Configure these pipelines to automatically classify and mask data based on metadata rules at publish time.
Use APIs or custom functions in the processing logic to enforce policies.

Monitoring & Auditing

Set up logging mechanisms to capture policy enforcement events.
Use monitoring tools like Amazon CloudWatch, Prometheus, or Grafana to track compliance over time.
Implement automated alerts for non-compliance and generate detailed audit reports.

#### Closing Summary

By leveraging a centralized metadata management system and automated enforcement mechanisms, you can ensure that PII classification and masking policies are applied uniformly across all domains in a Data Mesh architecture. This approach not only ensures consistency but also enhances data security and privacy compliance.

#### Common Mistakes/Follow-Ups
Lack of Centralized Metadata Management: Without a single source of truth for metadata, it’s challenging to enforce consistent policies.
Manual Policy Enforcement: Manual processes can lead to inconsistencies and errors.
Insufficient Monitoring: Lack of continuous monitoring can result in undetected policy violations.

Common follow-ups might include:

How do you ensure data consistency when multiple domains are involved?
What strategies would you use to handle edge cases, such as legacy data or new types of PII?
How do you balance security and performance in your enforcement mechanisms?

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    METADATA ||--o{ PII_RULES : contains
    TABLES {
        +--|> MASKING_POLICIES
    }
```

**Code:**

```python
# Example Python code for policy enforcement using Apache Flink

from pyflink.dataset import ExecutionEnvironment
from pyflink.table import BatchTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = BatchTableEnvironment.create(env)

# Define PII classification rules
pii_rules = {
    "SSN": r"\d{3}-\d{2}-\d{4}",
    "Name": r"[A-Z][a-z]+"
}

# Load metadata from a CSV file
t_env.connect(FileSystem().path('/path/to/metadata'))
     .with_format(OldCsv()
                  .field('name', DataTypes.STRING())
                  .field('rules', DataTypes.MAP(DataTypes.STRING(), DataTypes.STRING())))
     .with_schema(Schema()
                  .field('name', DataTypes.STRING())
                  .field('rules', DataTypes.MAP(DataTypes.STRING(), DataTypes.STRING()))
     ).register_table_source('metadata_source')

# Apply policies in the table environment
t_env.scan('metadata_source') \
     .filter("name in ('SSN', 'Name')") \
     .map(lambda t: (t.name, re.sub(pii_rules[t.name], '****', t.rules))) \
     .print()
```

**Example input / output:**

```text
Input:
Metadata file (metadata.csv):
name, rules
"SSN", \d{3}-\d{2}-\d{4}
"Name", [A-Z][a-z]+

Output:
(name, masked_rules)
(SSN, ****-****-****)
(Name, *********)
```

**Data:**

```json
{
  "PII_Rules": {
    "SSN": r"\d{3}-\d{2}-\d{4}",
    "Name": r"[A-Z][a-z]+"
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Source
4. Extract Metadata
5. Is PII?
6. Mask Data
7. Publish Data
8. Log Compliance Event
9. Audit Report
10. Handle Edge Cases

