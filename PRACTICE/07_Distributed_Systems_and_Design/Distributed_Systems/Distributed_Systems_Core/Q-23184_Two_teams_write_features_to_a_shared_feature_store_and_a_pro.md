---
type: "practice"
difficulty: "Intermediate"
---

# Two teams write features to a shared feature store and a producer's silent schema change broke a downstream model in production. Explain how you'd version features and enforce a data contract so a producer can't silently poison consumers.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To prevent silent schema changes from breaking downstream models, we need to implement robust feature versioning and enforce a strict data contract. This involves using semantic versioning for features, ensuring backward compatibility, and implementing a validation layer that checks incoming data against the expected schema.

#### Triaging the Question

The core issue here is maintaining data integrity in a distributed system where multiple producers write to a shared feature store, and changes to the producer's schema must not silently break downstream models. The interviewer is probing your understanding of versioning strategies, schema validation, and best practices for managing changes in a feature store.

#### First-Principles Solution
1. Semantic Versioning

Semantic versioning (SemVer) allows us to manage versions with a major.minor.patch scheme:

Major: Breaking changes that require modifications in downstream consumers.
Minor: Backwards-compatible additions or changes.
Patch: Backwards-compatible bug fixes.

In the context of features, we can assign each feature a unique version number and tag it accordingly. For example, feature_x_v1, feature_y_v2.

2. Contract Enforcement

To enforce a data contract, we need to define and validate schemas at different stages:

Producer-side Validation: Ensure the producer writes data that adheres to the latest schema.

#### Cons
umer-side Validation: Before consuming, validate incoming data against the expected schema.

This can be achieved using tools like Apache Avro, Protobuf, or JSON Schema validation libraries.

3. Feature Store Design

A feature store should support multiple versions of features:

Versioned Storage: Store each version of a feature separately.
Metadata Management: Maintain metadata such as the latest version and schema definitions.

#### Example
Workflow

Producer Sends Data:

The producer writes data to the feature store, tagging it with a specific version.
Before sending, the producer performs validation against the current schema.

Feature Store Storage:

Upon receiving valid data, the feature store stores it in its respective versioned location.
Metadata is updated to reflect the new version and schema details.

#### Cons
umer Fetches Data:

#### Cons
umers fetch data based on their required version.
Before processing, consumers validate incoming data against their expected schema.

#### Implementation
Example
PYTHON
# Sample Python code for feature store with versioning

import avro.schema
from avro.datafile import DataFileReader, DataFileWriter
from avro.io import DatumReader, DatumWriter

class FeatureStore:
    def __init__(self):
        self.versioned_features = {}

    def write_feature(self, data, feature_name, version):
        schema = get_schema(version)  # Function to fetch schema by version
        writer = DataFileWriter(open(f'features_{feature_name}_{version}.avro', 'wb'), DatumWriter(), schema)
        writer.append(data)
        writer.close()

    def read_feature(self, feature_name, required_version):
        try:
            schema = get_schema(required_version)  # Function to fetch schema by version
            reader = DataFileReader(open(f'features_{feature_name}_{required_version}.avro', 'rb'), DatumReader())
            return next(reader)
        except StopIteration:
            raise ValueError("No data found for the required version")
        finally:
            reader.close()

def get_schema(version):
    # Fetch and return schema based on version
    pass

# Example usage
feature_store = FeatureStore()
data_to_write = {"name": "John", "age": 30}
feature_name = "user_profile"
version = "v1"
feature_store.write_feature(data_to_write, feature_name, version)

required_version = "v1"
data_read = feature_store.read_feature(feature_name, required_version)
print(data_read)

#### Closing Summary

By implementing semantic versioning and strict schema enforcement, we can ensure that changes to the producer’s schema do not silently break downstream models. This involves both producer-side validation before writing data and consumer-side validation when reading data.

#### Common Mistakes
and Follow-Ups
Ignoring Backwards Compatibility: Not considering how a new schema might affect existing consumers.
Lack of Metadata Management: Failing to maintain accurate metadata about feature versions and schemas.
Inadequate Validation Logic: Implementing validation only at one stage (either producer or consumer) without proper checks.

When asked follow-up questions, you can explore topics such as:

How to handle version mismatches between producers and consumers.
Strategies for rolling out new schema changes smoothly.
Tools and frameworks that facilitate feature store management in a distributed system.

**Comparison:**

| Version | Schema Definition               |
|---------|---------------------------------|
| v1      | { "name": string, "age": int }   |
| v2      | { "name": string, "age": int, "email": string } |

**Code:**

```python
import avro.schema
from avro.datafile import DataFileReader, DataFileWriter
from avro.io import DatumReader, DatumWriter

class FeatureStore:
    def __init__(self):
        self.versioned_features = {}

    def write_feature(self, data, feature_name, version):
        schema = get_schema(version)
        writer = DataFileWriter(open(f'features_{feature_name}_{version}.avro', 'wb'), DatumWriter(), schema)
        writer.append(data)
        writer.close()

    def read_feature(self, feature_name, required_version):
        try:
            schema = get_schema(required_version)
            reader = DataFileReader(open(f'features_{feature_name}_{required_version}.avro', 'rb'), DatumReader())
            return next(reader)
        except StopIteration:
            raise ValueError("No data found for the required version")
        finally:
            reader.close()

def get_schema(version):
    # Fetch and return schema based on version
    pass

# Example usage
feature_store = FeatureStore()
data_to_write = {"name": "John", "age": 30}
feature_name = "user_profile"
version = "v1"
feature_store.write_feature(data_to_write, feature_name, version)

required_version = "v1"
data_read = feature_store.read_feature(feature_name, required_version)
print(data_read)
```

**Example input / output:**

```text
# Example input and output

# Input: Writing data to the feature store
data_to_write = {"name": "John", "age": 30}

# Output: Data written successfully with version v1
feature_store.write_feature(data_to_write, "user_profile", "v1")

# Input: Reading data from the feature store with required version v1
required_version = "v1"

# Output: Data read and validated successfully
data_read = feature_store.read_feature("user_profile", required_version)
print(data_read)  # Output: {'name': 'John', 'age': 30}
```

**Data:**

```yaml
# Example schema definitions (using Avro)
user_profile_v1:
  type: record
  name: UserProfile
  fields:
    - {name: name, type: string}
    - {name: age, type: int}

user_profile_v2:
  type: record
  name: UserProfile
  fields:
    - {name: name, type: string}
    - {name: age, type: int}
    - {name: email, type: string}
```

**Flow:**

1. Validate Schema
2. Validate Schema
3. Producer
4. Write Data
5. Write to Feature Store
6. Feature Store
7. Consumer
8. Read Data
9. Consume Data
10. Process Data

