---
type: "practice"
difficulty: "Deep"
---

# How do you architect the ingestion tier so onboarding a new source is self-serve config, not an engineering project, while still mandatorily attaching schema, PII classification, ownership, and SLA at registration time? What's the paved-road contract, and how do you block ungoverned side-door ingestion?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Proving PII Residency](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an ingestion tier that supports self-serve onboarding of new data sources while ensuring mandatory schema, PII classification, ownership, and SLA registration at registration time, we need to design a robust and flexible system with clear governance controls. The core idea is to create a "paved-road contract" that ensures all necessary metadata is captured during the source registration process, reducing the complexity of onboarding new data sources to near-zero engineering effort.

#### Triage

The question probes our understanding of data architecture principles, particularly around self-service capabilities and governance compliance. Key assumptions include:

The system must support multiple data sources.
Self-serve onboarding implies minimal manual intervention for engineers after initial setup.
Mandatory metadata (schema, PII classification, ownership, SLA) ensures data integrity and security.

#### First-Principles Solution
Metadata Schema Definition: Define a standardized schema for metadata that includes fields like source_name, schema, PII_class, owner, and SLA.
Source Registration Interface: Create an API or CLI tool that allows users to register new data sources by providing the required metadata.
Automated Ingestion Pipeline: Develop a framework that automatically ingests data from registered sources, applying predefined transformations and validations based on the provided metadata.
Governance Controls: Implement mechanisms to enforce the collection of necessary metadata during registration and monitor compliance post-onboarding.
Step-by-Step Solution

Define Metadata Schema:

A JSON schema can be used for defining required fields and their types. For example:
JSON
{
  "source_name": "string",
  "schema": "object",
  "PII_class": "enum",
  "owner": "string",
  "SLA": "object"
}

Source Registration Interface:

Implement a REST API or CLI tool that allows users to register new sources by submitting the metadata JSON.

#### Example
API request:
HTTP
POST /sources/register
Content-Type: application/json

{
  "source_name": "sales_data",
  "schema": {
    "fields": [
      {"name": "id", "type": "integer"},
      {"name": "date", "type": "string"}
    ]
  },
  "PII_class": "none",
  "owner": "Sales Team",
  "SLA": {
    "max_lag_minutes": 60
  }
}

Automated Ingestion Pipeline:

The pipeline should validate the incoming data against the provided schema and PII classification.
If validation fails, appropriate actions (e.g., reject or log) should be taken.

#### Example
validation logic in Python:
PYTHON
def validate_source_data(data):
    required_fields = ["source_name", "schema", "PII_class", "owner", "SLA"]
    for field in required_fields:
        if field not in data:
            return False, f"Missing required field: {field}"
    # Additional validation logic...
    return True, None

Governance Controls:

Implement role-based access control (RBAC) to ensure that only authorized users can register new sources.
Use event-driven architectures to trigger notifications or automated actions based on SLA violations.

#### Closing Summary
and Common Mistakes/Follow-Ups

The key is to create a self-serve registration process while maintaining strict governance controls. Common pitfalls include:

Overlooking the importance of validation and enforcement mechanisms, leading to data quality issues.
Failing to implement robust error handling and logging, making it difficult to troubleshoot issues.

#### Follow-up
questions might include:

How would you handle data inconsistencies or schema changes after onboarding?
What considerations are there for scaling this system with a large number of sources?

**Comparison:**

```
Field	Type	Description
source_name	string	Unique name for the data source
schema	object	JSON schema defining fields and their types
PII_class	enum	Classification of personal identifiable information (e.g., none, sensitive)
owner	string	Name or team owning the source
SLA	object	Service Level Agreement for data latency and availability
```

**Code:**

```python
def validate_source_data(data):
    required_fields = ["source_name", "schema", "PII_class", "owner", "SLA"]
    for field in required_fields:
        if field not in data:
            return False, f"Missing required field: {field}"
    # Additional validation logic...
    return True, None

# Example usage
metadata = {
    "source_name": "sales_data",
    "schema": {
        "fields": [
            {"name": "id", "type": "integer"},
            {"name": "date", "type": "string"}
        ]
    },
    "PII_class": "none",
    "owner": "Sales Team",
    "SLA": {
        "max_lag_minutes": 60
    }
}

is_valid, error_message = validate_source_data(metadata)
if not is_valid:
    print(error_message)
else:
    print("Source data valid")
```

**Example input / output:**

```text
POST /sources/register

Request:
Content-Type: application/json

{
  "source_name": "sales_data",
  "schema": {
    "fields": [
      {"name": "id", "type": "integer"},
      {"name": "date", "type": "string"}
    ]
  },
  "PII_class": "none",
  "owner": "Sales Team",
  "SLA": {
    "max_lag_minutes": 60
  }
}

Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "success",
  "message": "Source registered successfully"
}
```

**Data:**

```json
{
  "source_name": "sales_data",
  "schema": {
    "fields": [
      {"name": "id", "type": "integer"},
      {"name": "date", "type": "string"}
    ]
  },
  "PII_class": "none",
  "owner": "Sales Team",
  "SLA": {
    "max_lag_minutes": 60
  }
}
```

**Flow:**

1. Yes
2. No
3. User submits metadata
4. Is required metadata present?
5. Register new source
6. Error message, resubmit withcorrect metadata
7. Data ingested and validatedaccording to schema
8. Monitor compliance andenforce SLA

