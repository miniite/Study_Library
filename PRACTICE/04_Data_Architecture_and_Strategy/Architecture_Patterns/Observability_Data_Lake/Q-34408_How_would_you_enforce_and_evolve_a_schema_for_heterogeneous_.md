---
type: "practice"
difficulty: "Deep"
---

# How would you enforce and evolve a schema for heterogeneous log sources landing in the lake so downstream ML features don't silently break?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Curated Observability Datasets](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Curated_Observability_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In an AIOps environment, ensuring that data from heterogeneous log sources lands consistently in a Data Lake is crucial to avoid silent failures when building ML models. The key is to design a flexible schema that can accommodate varying log structures while enforcing necessary constraints and evolving as new data sources are added or existing ones change.

#### Triaging the Question

The question revolves around two main aspects: enforcing a schema for logs from diverse sources and evolving this schema over time. The goal is to ensure that these logs can be processed consistently without breaking downstream ML models. The interviewer is probing your understanding of data ingesting pipelines, schema validation techniques, and strategies for schema evolution.

#### Assumptions
Logs are arriving in a Data Lake from multiple sources with varying structures.
These logs need to be transformed into a common format before feeding into ML models.
There will be an ongoing need to add or modify log sources over time.
The schema needs to be robust enough to handle future changes in data structure.
Step-by-Step Solution

Initial Schema Design:

Identify the essential fields across all current and known potential log sources. For example, timestamp, source identifier, and event type.
Define a base schema that includes these common fields.

Flexible Data Ingestion Pipeline:

Use tools like Apache NiFi or Kibana to ingest logs in their raw format.
Apply transformations using Apache Spark or Python scripts to standardize the data structure.

Schema Enforcement:

Implement a schema validation step that checks each log against the base schema during ingestion.
Use JSON Schema for flexible schema enforcement, allowing partial fields from different sources while enforcing mandatory ones.

Evolving the Schema:

Create an automated mechanism to detect changes in new or existing log sources.
Use tools like Delta Lake or Apache Iceberg to manage evolving schemas efficiently.
Implement a versioning system for schema changes, allowing multiple versions co-existing until transition is complete.

Monitoring and Feedback Loop:

Continuously monitor logs to catch any issues early using alerting mechanisms.
Develop a feedback loop where anomalies are logged and reviewed to identify schema violations or data quality issues.

#### Example
Schema Design
JSON
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "timestamp": { "type": "string" },
    "sourceIdentifier": { "type": "string" },
    "eventType": { "type": "string" },
    "optionalField1": { "type": ["null", "string"] },
    "optionalField2": { "type": ["null", "string"] }
  },
  "required": [ "timestamp", "sourceIdentifier", "eventType" ]
}
Evolution Mechanism
Version Control for Schemas: Maintain a versioned schema repository where new versions are periodically deployed.
Automated Testing: Run automated tests on the transformed data to ensure it meets expectations before moving to production.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
Ensuring consistent schema enforcement in a Data Lake with heterogeneous log sources requires a combination of initial design, flexible pipelines, robust validation mechanisms, strategic schema evolution, and continuous monitoring. This approach not only avoids silent failures but also supports the dynamic nature of modern data environments.

#### Common Mistakes/Follow-Ups

Over-restrictive Schemas: Failing to allow flexibility in optional fields can lead to breaking changes when new log sources are introduced.
Lack of Monitoring: Not having a robust monitoring system can result in undetected issues until they surface as errors in ML models.
Manual Schema Management: Relying on manual schema updates is prone to human error and can disrupt workflows.

#### Follow-ups
might include discussing specific tools or technologies used, trade-offs between flexibility and strictness, or strategies for handling large volumes of logs efficiently.

