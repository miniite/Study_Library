---
type: "practice"
difficulty: "Intermediate"
---

# How do you enforce that no producer can register a schema that breaks compatibility BEFORE it ships to prod — describe your CI gate.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Phased Schema Rollouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Phased_Schema_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce compatibility in a Schema Registry, especially before new schemas are deployed to production, we need to establish a robust CI (Continuous Integration) gating mechanism that checks schema evolution against existing versions. This involves several key steps:

Understanding Compatibility: We first define what constitutes backward and forward compatibility for our schemas.
Schema Versioning: Each schema version is uniquely identified and registered in the Schema Registry.
Validation Rules: Define rules to ensure new schemas do not break existing consumers.
Integration with CI/CD Pipelines: Embed these validation checks into our CI pipeline.

Let’s break this down step by step:

#### Triaging the Question

The core of the question is about ensuring that any new schema registration does not violate established compatibility standards before it reaches production. The interviewer is probing your understanding of versioning, compatibility rules, and integration with CI/CD pipelines.

#### First-Principles Solution
Schema Version Identification: Each schema should have a unique identifier (version) that is checked against existing versions.
Compatibility Rules Definition: Define which changes are backward-compatible versus incompatible. For example:
Adding new fields (forward compatible)
Removing optional fields (backwards compatible if they can be ignored by older consumers)
Schema Registry Setup: Use a Schema Registry like Confluent Schema Registry, Apache Avro, or any other that supports versioning and schema evolution.
CI Pipeline Integration:
Unit Tests for Schema Validations: Write unit tests to validate new schemas against existing ones.
Pre-Commit Hook: Integrate these validations as a pre-commit hook in the CI pipeline.
Detailed Steps

Define Compatibility Rules:

Yes
No
Initial Schema
Add Field?
Forward Compatible
Backwards Incompatible
Check with Registry
Promote to Prod
Reject Schemas

Schema Versioning and Registry:

YAML
# Example schema version in Schema Registry
{
  "schema": "{'type': 'record', 'name': 'User', 'fields': [{'name': 'id', 'type': 'int'}, {'name': 'name', 'type': 'string'}]}",
  "version": 1,
  "compatibility": "BACKWARD"
}

CI Pipeline Integration:

Integrate schema validation tools like confluent-schemaregistry-client or similar.

#### Example
pre-commit hook in a CI pipeline (using Jenkins, GitLab, etc.):
BASH
#!/bin/bash

# Paths to your schemas and the registry client
SCHEMA_PATH=/path/to/schemas
REGISTRY_CLIENT=jmeter

# List all current versions
CURRENT_VERSIONS=$($REGISTRY_CLIENT get-versions)

for SCHEMA in $SCHEMA_PATH/*.avsc; do
    VERSION=$(awk -F'[{}:, ]' '{print $3}' < $SCHEMA | grep version | head -1)
    if [[ ! " $CURRENT_VERSIONS " =~ " $VERSION " ]]; then
        $REGISTRY_CLIENT validate-schema $SCHEMA
    fi
done

if [ $? -eq 0 ]; then
    echo "Schema validation passed."
else
    echo "Schema validation failed. Aborting CI build."
    exit 1
fi

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that only compatible schemas are deployed to production, reducing the risk of breaking existing consumer services. Common mistakes include skipping compatibility checks, not properly versioning schemas, or misconfiguring the registry.

#### Follow-up
questions might explore:

How do you handle schema evolution in a microservices architecture?
What about handling different types of compatibility (backwards, forwards, both)?
How do you manage conflicts when multiple teams are working on evolving schemas?

**Comparison:**

```
Schema Version	Compatibility Type
1	BACKWARD
2	FORWARD
3	BOTH
```

**Code:**

```bash
#!/bin/bash

# Paths to your schemas and the registry client
SCHEMA_PATH=/path/to/schemas
REGISTRY_CLIENT=jmeter

# List all current versions
CURRENT_VERSIONS=$($REGISTRY_CLIENT get-versions)

for SCHEMA in $SCHEMA_PATH/*.avsc; do
    VERSION=$(awk -F'[{}:, ]' '{print $3}' < $SCHEMA | grep version | head -1)
    if [[ ! " $CURRENT_VERSIONS " =~ " $VERSION " ]]; then
        $REGISTRY_CLIENT validate-schema $SCHEMA
    fi
done

if [ $? -eq 0 ]; then
    echo "Schema validation passed."
else
    echo "Schema validation failed. Aborting CI build."
    exit 1
fi
```

**Example input / output:**

```text
$ bash validate_schema.sh
Schema validation passed.
```

**Data:**

```yaml
# Example schema version in Schema Registry
{
  "schema": "{'type': 'record', 'name': 'User', 'fields': [{'name': 'id', 'type': 'int'}, {'name': 'name', 'type': 'string'}]}",
  "version": 1,
  "compatibility": "BACKWARD"
}
```

**Flow:**

1. Yes
2. No
3. Initial Schema
4. Add Field?
5. Forward Compatible
6. Backwards Incompatible
7. Check with Registry
8. Promote to Prod
9. Reject Schemas

