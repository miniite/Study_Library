---
type: "practice"
difficulty: "Expert"
---

# Design a schema-evolution and data-contract governance model between producers and 20 consumer teams so that a producer can ship a change on Tuesday without a war-room. Who owns the contract and how is it enforced in CI?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Composition and BFF](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust schema-evolution and data-contract governance model, we need to ensure that producers can make changes confidently while maintaining backward compatibility with consumers across 20 teams. The core of this solution involves defining clear ownership and enforcement mechanisms within the Continuous Integration (CI) pipeline.

First-principles thinking starts by recognizing that backwards compatibility is crucial for seamless deployments without disrupting consumer teams. This requires a well-defined contract between producers and consumers, ensuring that any change in schema or data structure does not break existing consumers. The producer must own this contract, as they are the ones making changes to the system.

#### Triaging the Question

The question asks us to design a schema-evolution and data-contract governance model where:

Schema Evolution: Allows producers to update their schemas while ensuring compatibility.
Data Contract Governance: Ensures that consumers can trust the incoming data from producers.
CI Enforcement: Integrates these mechanisms within the CI pipeline.

#### Assumptions
There are 20 consumer teams consuming data from a single producer.
The producer updates its schema and data contracts frequently.

#### Cons
umers rely on the producer’s data to function correctly, without any interruptions during upgrades.
The solution must be scalable and maintainable.
Step-by-Step Solution

Schema Versioning:

Producers should version their schemas. Each new release can have a version number (e.g., v2), which indicates the schema format of the data being produced.

#### Cons
umers can check for schema compatibility during runtime, ensuring that they only process data from compatible versions.

Data Contracts:

Define strict contracts governing how and what data is exchanged between producer and consumer.
Use metadata to describe each piece of data, including its type, format, and constraints (e.g., JSON schemas).

CI Pipeline Integration:

Integrate schema versioning and data contract validation into the CI pipeline.
Automatically run tests for backward compatibility before merging changes.

#### Implementation

Schema Versioning:

PYTHON
# Example Python code to define a schema version
def produce_data(version, *args):
    if version == 'v2':
        return {"key": "value", "type": "string"}
    elif version == 'v3':
        return {"key1": 123, "key2": ["a", "b", "c"]}

Data Contracts:

YAML
# Sample YAML contract for data
schema_version: v2
fields:
  - name: key
    type: string

#### cons
traints: [required]
  - name: type
    type: string

#### cons
traints: [in: ["string", "number"]]

CI Pipeline:

YAML
steps:
  - run-tests:
      script: |
        python test_compatibility.py
  - deploy:
      script: |
        git push origin main

#### Closing Summary
and Common Mistakes

To summarize, the key is to ensure that schema changes are managed with versioning, data contracts are strictly enforced, and CI integrates these checks. The producer owns the contract and enforces it in CI by running compatibility tests.

#### Common Mistakes

Failing to define clear schema versions.
Overlooking runtime validation of incoming data.
Not integrating schema and contract checks into CI.
Ignoring edge cases where data contracts might break unexpectedly.

#### Follow-Ups
How do you handle breaking changes in the schema?
What mechanisms do you have for rollback if a change breaks consumers?
How does this model scale as more teams consume from the producer?

