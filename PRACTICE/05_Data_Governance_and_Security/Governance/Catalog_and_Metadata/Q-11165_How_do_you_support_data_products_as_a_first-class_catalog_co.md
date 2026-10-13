---
type: "practice"
difficulty: "Expert"
---

# How do you support 'data products' as a first-class catalog concept in a mesh — bundling schema, SLAs, owner, quality, docs, and access — and what metadata does a consumer need to trust and depend on a product?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cataloging External Data](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Cataloging_External_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To support 'data products' as first-class catalog concepts in a mesh architecture, we need to create an intelligent, dynamic data catalog that aggregates schema, Service Level Agreements (SLAs), owners, quality metadata, documentation, and access controls into a cohesive product. Consumers rely on this catalog for trust and dependency, ensuring they can make informed decisions about the data's fitness for use.

The core idea is to design a metadata-driven approach where each 'data product' is treated as a self-describing entity with its own metadata profile that captures all necessary attributes. This involves integrating advanced metadata engineering practices, leveraging semantic layers, and employing robust validation mechanisms.

#### Triage
What the Question Is Asking

The question probes how to build a comprehensive data catalog system capable of supporting 'data products' by bundling essential metadata such as schema, SLAs, ownership details, quality metrics, documentation, and access controls. It also seeks to ensure that consumers can trust and depend on these products based on the provided metadata.

#### Assumptions
The data mesh architecture is in place.
We are using a modern metadata management platform.

#### Cons
umers have defined criteria for trusting data products.

#### First-Principles Solution
1. Metadata Model

Define a comprehensive metadata model that includes:

Schema: Data structure and types.
SLAs: Performance, availability, and response time guarantees.
Owner/Responsibility: Person or team responsible for the product.
Quality Metrics: Data freshness, completeness, consistency.
Documentation: User guides, API documentation, data dictionary.
Access Control Policies: Who can access what and under what conditions.
2. Catalog Architecture

Design a catalog architecture that:

Integrates metadata from various sources (data lakes, warehouses, APIs).
Supports real-time updates through event-driven mechanisms.
Provides a unified view of all 'data products'.
3. Validation Mechanisms

Implement validation checks to ensure the quality and integrity of data products:

Data lineage tracking for provenance and change history.
Automated tests for schema compliance and SLA adherence.
4. Metadata Consumption

Ensure consumers can easily access and interpret metadata by:

Providing a searchable interface (e.g., GraphQL, REST API).
Generating interactive documentation using tools like Swagger UI or ReDoc.
Implementing permission controls based on roles and responsibilities.
5. Continuous Improvement

Establish processes for continuously improving the catalog and metadata quality through:

Regular audits of data products.
Feedback loops with users and stakeholders.
Iterative refinement of the metadata model.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, supporting 'data products' in a mesh architecture requires a robust metadata-driven approach that encapsulates all necessary information about the product. This involves defining a comprehensive metadata model, designing an efficient catalog architecture, implementing rigorous validation mechanisms, and ensuring metadata is easily consumable by stakeholders.

#### Common mistakes
include:

Overlooking critical metadata elements like quality metrics.
Failing to integrate real-time updates effectively.
Not providing sufficient documentation or access controls.
Ignoring feedback and iterative improvements.

#### Follow-ups
might explore specific technologies used (e.g., AWS Glue, Azure Data Catalog) or discuss potential challenges in scaling such a system.

**Comparison:**

| Metadata Element | Description |
|------------------|-------------|
| Schema           | Defines the structure and types of data fields. |
| SLAs             | Specifies performance, availability, and response time guarantees. |
| Owner/Responsibility | Identifies who is responsible for maintaining the data product. |
| Quality Metrics  | Measures data freshness, completeness, and consistency. |
| Documentation    | Provides user guides, API documentation, and data dictionary. |
| Access Control Policies | Controls who can access what data and under what conditions. |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Metadata Model] --> B[Schema];
    A --> C[SLAs];
    A --> D[Owner/Responsibility];
    A --> E[Quality Metrics];
    A --> F[Documentation];
    A --> G[Access Control Policies];

    B --> H[Data Structure];
    C --> I[Performance];
    D --> J[Tenants];
    E --> K[Freshness, Completeness, Consistency];
    F --> L[User Guides, API Docs];
    G --> M[Roles & Permissions];
```

**Example input / output:**

```text
Input:
- A request for data product 'SalesData'

Output:
{
  "schema": {
    "fields": [
      { "name": "id", "type": "integer" },
      { "name": "amount", "type": "float" }
    ]
  },
  "slas": {
    "performance": 10,
    "availability": 99.9
  },
  "owner": "SalesTeam",
  "qualityMetrics": {
    "freshness": 3600,
    "completeness": 85
  },
  "documentation": {
    "userGuideURL": "https://docs.example.com/salesguide",
    "apidoc": {
      "operations": [
        { "method": "GET", "path": "/api/salesdata" }
      ]
    }
  },
  "accessPolicies": [
    { "role": "admin", "permissions": [ "read", "write" ] },
    { "role": "user", "permissions": [ "read" ] }
  ]
}
```

**Data:**

```json
{
  "schema": {
    "fields": [
      { "name": "id", "type": "integer" },
      { "name": "name", "type": "string" }
    ]
  },
  "slas": {
    "performance": 10,
    "availability": 99.9
  },
  "owner": "DataEngineeringTeam",
  "qualityMetrics": {
    "freshness": 3600,
    "completeness": 85
  },
  "documentation": {
    "userGuideURL": "https://docs.example.com/guide",
    "apidoc": {
      "operations": [
        { "method": "GET", "path": "/api/data" }
      ]
    }
  },
  "accessPolicies": [
    { "role": "admin", "permissions": [ "read", "write" ] },
    { "role": "user", "permissions": [ "read" ] }
  ]
}
```

**Flow:**

1. Metadata Source
2. Aggregation Layer
3. METADATA DATABASE
4. Search Interface
5. Consumer
6. Data Product Request
7. Metadata Lookup
8. Validate SLA
9. Check Quality Metrics
10. Permission Check
11. Provide Data Product

