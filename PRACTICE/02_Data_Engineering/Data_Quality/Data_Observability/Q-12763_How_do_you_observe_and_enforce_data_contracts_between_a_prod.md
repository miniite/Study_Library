---
type: "practice"
difficulty: "Deep"
---

# How do you observe and enforce data contracts between a producing microservice and your ingestion pipeline?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Quality Monitoring](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To observe and enforce data contracts between a producing microservice and an ingestion pipeline, we need to establish a robust monitoring framework that ensures both semantic and syntactic integrity of the data being produced and consumed. This involves setting up metrics, logs, and alerts; validating data against predefined schemas or rules; and implementing automated validation checks at various stages.

#### Triaging the Question

The core of this question is about ensuring consistent, reliable data transfer between a microservice (producer) and an ingestion pipeline. The interviewer likely probes your understanding of data contract enforcement mechanisms, monitoring strategies, and potential failure points in the data pipeline. We need to assume that the producer service generates structured or semi-structured data, and it’s crucial to validate this data as part of its lifecycle.

#### First Principles Solution
Define Data Contracts: Clearly define what constitutes valid data for both the microservice and the ingestion pipeline.
Schema Validation: Implement schema validation at the source (producer) and destination (ingestion pipeline).
Data Transformation: Use transformations to ensure that any necessary data cleaning or formatting is applied before storage.
Monitoring and Alerts: Set up monitoring and alert systems to notify stakeholders of anomalies in data flow.
Step-by-Step Solution

Schema Definition

Define the expected schema for the data using JSON Schema, Avro, or another format suitable for your environment.

Producer Side Validation

Implement validation logic within the microservice to ensure that only valid data is emitted. This can be done in a middleware layer before sending the data downstream.

Ingestion Pipeline Validation

At the ingestion pipeline’s entry point, validate incoming data against the defined schema.
Use tools like Apache Beam or AWS Glue for processing and validation during ingestion.

Transformation and Normalization

If necessary, transform raw data into a normalized form before storage to ensure consistent structure across the pipeline.

Monitoring and Logging

Set up logging mechanisms to capture critical events such as schema violations, missing fields, or invalid values.
Configure alerts for any deviations from expected data patterns.

Continuous Integration/Continuous Deployment (CI/CD)

Integrate these validation checks into your CI/CD pipelines to ensure that any changes in the microservice’s output are validated before deployment.

#### Closing Summary
and Common Mistakes

This approach ensures that data is clean, consistent, and compliant with predefined contracts. By implementing schema validation at both producer and consumer ends, we can minimize issues like data corruption or unexpected formats. However, common pitfalls include insufficient validation logic, lack of comprehensive logging, and inadequate alerting mechanisms.

#### Follow-Up
Considerations
Performance Impact: Schema validation can introduce latency; consider trade-offs between performance and data quality.
Error Handling: Implement robust error handling strategies to gracefully deal with schema violations without disrupting the entire pipeline.
Granularity of Monitoring: Tailor monitoring granularity based on data volume and criticality.

**Comparison:**

```
Validation Type	Description
Schema Validation	Ensures data conforms to a predefined structure.
Transformation	Converts raw data into a consistent format.
Monitoring	Tracks and logs critical events.
```

**Code:**

```python
# Example Python code for schema validation using JSON Schema
import jsonschema
from json import loads

def validate_data(data, schema):
    try:
        jsonschema.validate(instance=loads(data), schema=schema)
    except jsonschema.exceptions.ValidationError as ve:
        return False, str(ve)
    return True, None

# Example schemas
data_schema = {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"}
    },
    "required": ["id", "name"]
}

# Usage in a microservice
producer_data = '{"id": 1, "name": "John Doe"}'
is_valid, error_msg = validate_data(producer_data, data_schema)
if not is_valid:
    print(f"Validation failed: {error_msg}")
```

**Example input / output:**

```text
Input:
Producer Data: '{"id": 1, "name": "John Doe"}'
Schema: {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"}
    },
    "required": ["id", "name"]
}

Output:
Validation Result: True
```

**Data:**

```json
{
    "data_schema": {
        "type": "object",
        "properties": {
            "id": {"type": "integer"},
            "name": {"type": "string"}
        },
        "required": ["id", "name"]
    }
}
```

**Flow:**

1. Yes
2. No
3. Invalid Schema
4. Valid Schema
5. Producer
6. Schema Validation
7. Is Valid?
8. Send Data to IngestionPipeline
9. Alert Developer andRetry/Correct Data
10. Ingestion Pipeline
11. Receive Data
12. Reject Data and Alert
13. Process and Store Data

