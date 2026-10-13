---
type: "practice"
difficulty: "Deep"
---

# How does a data contract (e.g. via a contract registry / dbt contracts) relate to a schema registry — are they the same thing and where do responsibilities differ?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Phased Schema Rollouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Phased_Schema_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Data Engineering context, both a data contract and a schema registry serve crucial roles in ensuring consistency and correctness of data across systems. However, while they overlap in their objectives, they operate on different levels and have distinct responsibilities.

A data contract typically refers to agreements or rules about the structure and integrity of data, often formalized using tools like dbt contracts or JSON Schema validation. These contracts ensure that when a model is transformed from raw data sources to analytics or operational systems, the resulting dataset adheres to predefined standards.

On the other hand, a schema registry is a more technical component used in distributed and microservices-based architectures. It manages versions of data schemas in a centralized manner, ensuring that all services consuming or producing data use the correct schema at any given time. The schema registry helps maintain compatibility between producers and consumers by tracking changes to data structures over time.

#### Triage
What the Question is Probing

The question aims to clarify the relationship between these two concepts—whether they are the same, how they differ in their roles, and where their responsibilities overlap or diverge. We need to establish clear boundaries around what each concept covers to avoid misunderstandings and ensure effective data management.

#### First-Principles Solution

Data Contracts:

Focus on logical definitions: These are the rules that define how a piece of data should be structured, formatted, or transformed.
Tools like dbt contracts allow you to specify these rules in code. For example, dbt validate can check if your models conform to certain validation rules (e.g., required fields, regex constraints).

Schema Registry:

Focus on physical definitions: These are the actual data formats and structures used by systems to transmit and store data.
Schema registries like Confluent Schema Registry or Apache Avro ensure that every piece of data conforms to a specific schema version. They help manage schema evolution over time, ensuring backward compatibility.
Key Differences

Purpose:

Data contracts are about defining the structure of data at the logical level.
Schema registries focus on managing and validating the physical structures used in data transmission.

Scope:

Data contracts can span multiple stages of a pipeline (e.g., ETL, transformations).
Schema registries operate primarily within distributed systems and microservices architectures.

#### Implementation
Tools:

Data contracts often use schema languages like JSON Schema or YML files.
Schema registries are typically implemented using specific software solutions with APIs for registration, retrieval, and validation of schemas.

#### Closing Summary

In summary, while both data contracts and schema registries serve important purposes in data engineering, they operate on different levels. Data contracts focus on defining the logical structure and integrity of data within your transformations, whereas schema registries manage physical schema versions to ensure compatibility across distributed systems. Understanding their roles can help you design robust data pipelines that are both flexible and reliable.

#### Common Mistakes/Follow-Ups

Mistake: Assuming a data contract replaces the need for a schema registry.

#### Follow-Up
A data contract is just one part of a broader strategy; it doesn't cover all aspects of schema management, especially in distributed environments.

Mistake: Treating schema registries as optional tools only used in specific architectures.

#### Follow-Up
Even monolithic applications benefit from clear and versioned schemas, which can be managed by a schema registry.

**Comparison:**

```
Schema Registry Feature	Description
Schema Versioning	Manages schema evolution while ensuring backward compatibility.
Registration	Registers schemas in a centralized manner, allowing easy retrieval and updates.
Validation	Validates incoming data against registered schemas to ensure consistency.
```

**Code:**

```python
# Example of dbt contract validation
from dbt.contracts.validation import ValidationException

try:
    # Assuming a model `sales_data` with validation rules defined in the model's metadata
    dbt.validate_model('sales_data')
except ValidationException as e:
    print(f"Validation failed: {e}")
```

**Example input / output:**

```text
Input:
- Raw data from a CSV file with missing product IDs.
Output:
- Validation failure due to missing required fields; logs indicate validation rules were applied correctly.
```

**Data:**

```json
{
  "schema": {
    "type": "object",
    "properties": {
      "product_id": {"type": "integer"},
      "name": {"type": "string"},
      "price": {"type": "number"}
    },
    "required": ["product_id", "name"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Contract
6. Does it define logicalstructure?
7. Dbt Contracts
8. Schema Registry
9. Schema Registry
10. Is it for physical schemamanagement?
11. Confluent Schema Registry
12. Apache Avro Registry

