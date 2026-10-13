---
type: "practice"
difficulty: "Deep"
---

# How do you reconcile the fact that a schema can be registry-compatible but still violate a business data contract (e.g. new nullable field that must never be null in practice)?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Phased Schema Rollouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Phased_Schema_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When dealing with schema registries and evolving schemas, it's crucial to ensure that changes align with both technical requirements and business logic. However, there are scenarios where a new schema might be compatible from a technical perspective but violate critical business rules. For instance, adding a nullable field that must never be null in practice can pose significant issues. This problem requires a layered approach involving schema validation, metadata management, and automated enforcement mechanisms.

#### Triaging the Question

The core of this question revolves around reconciling schema compatibility with strict business constraints. Specifically:

Technical Compatibility: A new schema is registered in the registry but adheres to technical standards.
Business Compliance: The schema change introduces a violation of existing business rules, such as mandatory non-null fields.

#### Assumptions
We are using a well-established schema registry (e.g., Apache Avro or Kafka Connect Schema Registry).
There is an existing data pipeline that processes the data with dependencies on these schemas.
The business has strict policies about certain field values, such as mandatory non-null fields.

#### Solution
from First Principles:
Step 1: Understanding Schema Compatibility

A schema in a registry like Apache Avro or Kafka Connect is considered compatible if it adheres to predefined rules (e.g., backward compatibility). However, this doesn't always align with business logic. For example:

Schema Registered
Registry Compatible
Mandatory Non-Null Field
Business Violation
Step 2: Schema Validation at the Data Pipeline Level

To ensure that a schema change does not violate business rules, we need to implement additional validation steps in our data pipeline:

PYTHON
def validate_schema(some_record):
    # Check for mandatory non-null fields
    if some_record['mandatory_field'] is None:
        raise ValueError("Mandatory field cannot be null")
Step 3: Metadata Management and Documentation

Maintain detailed metadata about the schema changes, including business rules, in a centralized repository. This helps traceability and compliance checks.

JSON
{
    "schema_id": "v2",
    "fields": [
        { "name": "mandatory_field", "type": "string" },
        { "name": "optional_field", "type": "null" }
    ],
    "business_rules": {
        "non_null_fields": ["mandatory_field"]
    }
}
Step 4: Automated Enforcement Mechanisms

Use automated tools or scripts to enforce business rules during data processing. For instance, integrating schema validation into our ETL processes:

PYTHON
def process_data(records):
    for record in records:
        validate_schema(record)
        # Process the record
        process_record(record)
Step 5: Auditing and Monitoring

Implement continuous monitoring to detect and flag any violations of business rules. Use alerts or dashboards to notify stakeholders.

PLAINTEXT
- Monitor log files for errors related to schema validation.
- Set up automated checks during nightly batch jobs.

#### Closing Summary

In summary, while a schema might be technically compatible in the registry, it's essential to implement additional layers of validation and enforcement mechanisms to ensure that business rules are not violated. This involves detailed metadata management, robust validation logic, and continuous monitoring.

#### Common Mistakes/Follow-Ups
Over-relying on Schema Registry: Assuming technical compatibility automatically ensures business compliance.
Ignoring Business Rules in Schema Evolution: Failing to document and enforce critical business policies during schema changes.
Lack of Monitoring: Not having automated checks or alerts for violations can lead to undetected issues.

**Comparison:**

```
Schema Field	Type	Business Rule
mandatory_field	string	Non-null
optional_field	null	Optional
```

**Code:**

```python
def validate_schema(some_record):
    if some_record['mandatory_field'] is None:
        raise ValueError("Mandatory field cannot be null")

def process_data(records):
    for record in records:
        validate_schema(record)
        # Process the record
        process_record(record)

class SchemaMetadata:
    def __init__(self, schema_id, fields, business_rules):
        self.schema_id = schema_id
        self.fields = fields
        self.business_rules = business_rules

# Example usage
schema_metadata = SchemaMetadata(
    "v2",
    [
        {"name": "mandatory_field", "type": "string"},
        {"name": "optional_field", "type": "null"}
    ],
    {
        "non_null_fields": ["mandatory_field"]
    }
)
```

**Example input / output:**

```text
Input: {"mandatory_field": null, "optional_field": "value"}
Output: ValueError("Mandatory field cannot be null")

Input: {"mandatory_field": "valid_value", "optional_field": "value"}
Output: Processed successfully
```

**Data:**

```json
{
    "schema_id": "v2",
    "fields": [
        { "name": "mandatory_field", "type": "string" },
        { "name": "optional_field", "type": "null" }
    ],
    "business_rules": {
        "non_null_fields": ["mandatory_field"]
    }
}
```

**Flow:**

1. Pass
2. Fail
3. Receive Record
4. Validate Mandatory Fields
5. Process Record
6. Handle Error

