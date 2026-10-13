---
type: "practice"
difficulty: "Deep"
---

# How would you deploy a schema change to an Avro-serialized Kafka topic through CI so the schema registry's compatibility mode blocks a producer change that would break existing consumers?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in Event Driven Deploys](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Schema_Evolution_in_Event_Driven_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to ensure that any schema changes in our Avro-serialized Kafka topics are deployed safely and with strict backward-compatibility checks enabled by the Schema Registry’s compatibility mode. This involves a combination of CI/CD practices to automate testing and deployment while adhering to the necessary constraints.

#### Triage

The core question is about deploying schema changes through CI such that:

The schema change process is automated.
Compatibility mode in the Schema Registry blocks changes that would break existing consumers.
Changes do not inadvertently introduce breaking changes into production.

#### Assumptions
and Context
We have a Kafka cluster with Avro serialization for our topics.
A Schema Registry service is used to manage and version control schemas.
The application uses consumers and producers interacting with these Kafka topics.
CI/CD pipelines are already in place, and we need to integrate schema change validation into them.

#### First Principles Solution

The solution involves the following steps:

Schema Definition: Define the Avro schema for your data.
Committing Schemas to Registry: Use Schema Registry APIs to commit schemas with appropriate versioning.
Setting Compatibility Levels: Ensure that the Schema Registry is configured to enforce backward and forward compatibility modes as needed.
CI Pipeline Integration: Automate the process of committing new schemas through CI/CD, ensuring tests are run before any schema updates.
Validation Tests: Implement unit tests and integration tests to validate the compatibility of the new schema with existing consumers.
Step-by-Step Solution

Schema Definition

Define your Avro schema using a .avsc file or directly in your application code.

Committing Schemas to Registry

PYTHON
from confluent_kafka.schema_registry import SchemaRegistryClient, AvroSerializer

# Initialize the Schema Registry client with your registry URL and API key
schema_registry_conf = {'url': 'http://localhost:8081'}
schema_registry_client = SchemaRegistryClient(schema_registry_conf)

# Define the avro schema
avro_schema_str = """
{
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "int"},
        {"name": "name", "type": "string"}
    ]
}
"""

# Register the schema with a specific subject name
topic = 'user-topic'
subject_name = f'{topic}-value'
avro_serializer = AvroSerializer(schema_registry_client, subject_name)

# Commit the new version of the schema to the registry
schema_id = schema_registry_client.register(subject_name, avro_schema_str)

Setting Compatibility Levels

Configure the Schema Registry to enforce compatibility modes.
JSON
{
  "compatibility": "BACKWARD",  # or FORWARD, FULL
  "schema.registry.url": "http://localhost:8081"
}

CI Pipeline Integration

Integrate schema change validation into your CI pipeline using tools like Jenkins, GitHub Actions, GitLab CI.
YAML
jobs:
  - job: validate_schema_change
    steps:
      - script: |
          # Run tests to ensure the new schema does not break existing consumers
          python test_schema_compatibility.py --new-schema-path=/path/to/new/schema
        when: on_success

Validation Tests

Implement unit and integration tests to validate the compatibility of the new schema.
PYTHON
import pytest

@pytest.mark.parametrize("schema_path", ["/path/to/new/schema"])
def test_schema_compatibility(schema_path):
    # Load the new schema from file
    with open(schema_path, 'r') as f:
        new_schema = json.load(f)

    # Connect to Kafka and Schema Registry
    kafka_config = {'bootstrap.servers': KAFKA_BOOTSTRAP_SERVERS}
    sr_config = {'url': SCHEMA_REGISTRY_URL}

    # Validate the new schema against existing consumers
    validate_consumer_compatibility(kafka_config, sr_config, new_schema)

def validate_consumer_compatibility(kafka_config, sr_config, new_schema):
    # Implement logic to connect to existing consumers and test compatibility
    pass

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the key is to integrate schema change validation into your CI process while ensuring that the Schema Registry's compatibility mode enforces backward and forward compatibility. Common mistakes include neglecting to validate new schemas before deployment or not properly configuring the Schema Registry for strict compatibility checks.

#### Follow-up
questions might include:

How do you handle breaking changes in a controlled manner?
What tools and libraries are used for schema validation?
How do you manage multiple versions of schemas in your application?

**Code:**

```python
from confluent_kafka.schema_registry import SchemaRegistryClient, AvroSerializer

# Code to initialize the Schema Registry client
schema_registry_conf = {'url': 'http://localhost:8081'}
schema_registry_client = SchemaRegistryClient(schema_registry_conf)

# Code to register a new schema with the registry
avro_schema_str = """
{
    "type": "record",
    "name": "User",
    "fields": [
        {"name": "id", "type": "int"},
        {"name": "name", "type": "string"}
    ]
}
"""
subject_name = 'user-topic-value'
schema_id = schema_registry_client.register(subject_name, avro_schema_str)
```

**Data:**

```json
{
  "compatibility": "BACKWARD",
  "schema.registry.url": "http://localhost:8081"
}
```

**Flow:**

1. CI Pipeline Starts
2. Schema Definition
3. Commit Schema to Registry
4. Set Compatibility Level
5. Integrate Validation Tests
6. Deploy to Production
7. Monitor for Issues

