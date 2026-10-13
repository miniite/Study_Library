---
type: "practice"
difficulty: "Expert"
---

# How do you architect discoverability so a data product surfaces its trust signals like quality score, ownership, freshness, and SLA at the point of consumption?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product Discoverability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Product_Discoverability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Data Mesh architecture, the key to achieving high discoverability is ensuring that metadata about datasets, including their quality scores, ownership, freshness, and service level agreements (SLAs), are readily available and easily accessible at the point of consumption. This involves creating a robust metadata layer that surfaces these trust signals in an intuitive and actionable manner. The core idea here is to build a federated metadata catalog that can be queried independently by data consumers.

To architect discoverability, we need to consider how different stakeholders—such as data producers, stewards, and consumers—interact with the data mesh. We must ensure that trust signals are not just available but also presented in a manner that is meaningful and actionable. The architecture will involve several steps: defining metadata standards, building a federated catalog, integrating with consumer tools, and continuously monitoring and improving.

**Comparison:**

```
Metadata Dimension	Data Point	Description
Quality	Score	Measure of data accuracy and reliability
Ownership	Name	Individual or team responsible for the dataset
Freshness	Date	Last time the dataset was updated
SLA	Availability	Guarantee on availability of the data
	Latency	Time taken to retrieve the data
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Metadata Definition] --> B{Are Metadata Standards Defined?};
    B -- Yes --> C[Define Metadata Schema];
    B -- No --> D[Establish Common Data Governance Policies];
    C --> E[Federate Metadata Across Data Lakes & Warehouses];
    E --> F[Develop API for Catalog Access];
    F --> G[Integrate with Consumer Tools (e.g., BI, ML Platforms)];
    G --> H[Monitor and Improve Metadata Quality];
```

**Example input / output:**

```

Given a dataset with the following metadata:

JSON
Copy
{
  "qualityScore": 0.85,
  "owner": "John Doe",
  "freshness": "2023-10-04T10:00:00Z",
  "sla": {
    "availability": "99.9%",
    "latency": "<=10s"
  },
  "comments": "Data refreshed hourly; recent changes made on 2023-10-04."
}

A consumer tool would present this as:

MARKDOWN
Copy
### Data Product Metadata

**Quality Score:** 85%
**Owner:** John Doe
**Freshness:** 2023-10-04T10:00:00Z
**SLA Availability:** 99.9%
**SLA Latency:** <=10s
```

**Data:**

```json
{
  "metadataSchema": {
    "qualityScore": "float",
    "owner": "string",
    "freshness": "datetime",
    "sla": "object",
    "comments": "string"
  }
}
```

**Flow:**

1. Data Producer
2. Steward
3. Define Metadata
4. Contribute Data to Mesh
5. Metadata Federated AcrossCatalogs
6. Consumer Tool Integration
7. Metadata Query & Display
8. Monitor and Update

