---
type: "practice"
difficulty: "Expert"
---

# In a data mesh, how do you keep lineage consistent when each domain publishes its own data products independently?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lineage for Dynamic SQL](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Lineage_for_Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In a data mesh architecture, ensuring lineage consistency is crucial to maintain trust and transparency in data usage across domains. The key idea is to implement a robust metadata management system that captures lineage at each domain level while also maintaining a central registry for cross-domain visibility.

#### Triaging the Question

The question asks how to keep lineage consistent when each domain independently publishes its own data products in a data mesh architecture. The core of this problem revolves around ensuring that the provenance and history of data are accurately captured, even as different domains manage their own data independently. The interviewer is probing your understanding of data governance principles, metadata management strategies, and the architectural implications for maintaining lineage.

#### Assumptions
Data Mesh Architecture: Each domain owns its data and services, with independent governance.
Lineage Capture: Lineage refers to the history of how data was created, transformed, and used.
Metadata Management: A central metadata system is available to capture and store lineage information.
Step-by-Step Solution
1. Establish a Metadata Schema

Define a comprehensive metadata schema that captures all necessary lineage attributes. This should include:

Data origin (source systems)
Transformation steps and scripts
Sensitive data tagging
Ownership details (domain, responsible parties)
Data Origin
Transformation Steps
Sensitive Data Tagging
Ownership Details
2. Implement Domain-Specific Metadata Stores

Each domain should have its own metadata store to capture lineage information locally:

Domain A: Manages product X with transformations T1 and T2.
Domain B: Manages product Y with transformations T3, T4, and T5.
Domain A
Product X
T1
T2
Domain B
Product Y
T3
T4
T5
3. Centralized Lineage Registry

Create a centralized lineage registry that aggregates metadata from all domains:

Registry: Serves as the single source of truth for cross-domain lineage.
Data Ingestion: Metadata from each domain is ingested into the central registry.
Domain A
Central Lineage Registry
Domain B
Audit Logs
4. Lineage Query and Reporting

Develop a lineage query engine that allows users to trace data flow across domains:

Query Engine: Supports complex lineage queries based on transformation steps, source systems, or ownership.
Visualization Tools: Provide dashboards for visualizing lineage paths.
User
Query Engine
Registry
Visualization Tools
5. Implement Data Provenance Policies

Ensure that data provenance policies are enforced to maintain consistency:

Policies: Define rules for how lineage information is captured, stored, and updated.
Automation: Use automation tools to enforce these policies across domains.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In a data mesh architecture, ensuring lineage consistency requires a combination of domain-specific metadata management and centralized registry. The key is to balance local autonomy with cross-domain visibility. Common mistakes include:

Over-reliance on central metadata systems without proper decentralization.
Insufficient automation leading to manual errors in capturing lineage information.

#### Follow-up
questions might explore how you would handle data lineage in a highly distributed system, the impact of real-time vs batch processing on lineage capture, or integration with existing governance frameworks like GDPR.

**Comparison:**

```
Metadata Store	Transformations
Domain A	[T1, T2]
Domain B	[T3]
Lineage Registry	Transformed Data Path
Domain A -> Product X	[source_system_1 -> T1 -> T2 -> Product X]
Domain B -> Product Y	[source_system_1 -> T3 -> Product Y]
```

**Code:**

```python
# Example metadata schema for domain-specific stores
class Metadata:
    def __init__(self, origin, transformations, sensitive_data, ownership):
        self.origin = origin
        self.transformations = transformations
        self.sensitive_data = sensitive_data
        self.ownership = ownership

# Example of a central lineage registry ingestion
def ingest_metadata(metadata_store, domain_name, metadata):
    # Ingest metadata from each domain into the central registry
    pass

# Example lineage query engine
def query_lineage(registry, query_params):
    # Execute complex lineage queries and return results
    pass
```

**Example input / output:**

```text
Input:
- Domain A metadata store with transformations T1 and T2
- Domain B metadata store with transformation T3

Output:
- Central lineage registry with full lineage information for all domains
```

**Data:**

```json
{
  "metadata_store": {
    "domain_a": [
      {"origin": "source_system_1", "transformations": ["T1", "T2"], "sensitive_data": true, "ownership": "team_x"},
      {"origin": "source_system_2", "transformations": ["T3"], "sensitive_data": false, "ownership": "team_y"}
    ],
    "domain_b": [
      {"origin": "source_system_1", "transformations": ["T4", "T5"], "sensitive_data": true, "ownership": "team_z"}
    ]
  },
  "central_registry": {
    "lineage": [
      {"origin": "source_system_2", "transformation_sequence": ["T3", "T4", "T5"], "sensitivity_tags": ["high"], "ownership": "team_x"}
    ]
  }
}
```

**Flow:**

1. User
2. Query Engine
3. Central Lineage Registry
4. Visualization Tools

