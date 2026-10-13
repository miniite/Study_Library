---
type: "practice"
difficulty: "Expert"
---

# A subsidiary in China must run on a local cloud with no cross-border transfer — how does that partition change your global catalog, lineage and metrics-layer design?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-cloud Data Contracts](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a multi-cloud data architecture where a subsidiary in China must use a local cloud service with strict adherence to data locality rules. This constraint significantly impacts how we structure our global catalog, lineage tracking, and metrics layer.

#### Triaging the Question

The primary concerns are:

Compliance: Ensuring all data remains within the bounds of Chinese regulations.
Global Access: Maintaining a unified view for stakeholders who need access to both local and remote data.
Performance and Latency: Minimizing latency for users in China while ensuring high performance for global operations.

#### Assumptions
The subsidiary operates entirely within China's borders, with no cross-border transfer allowed.
We maintain a central governance framework that enforces consistency across all regions.
Data lineage is crucial for auditing and compliance purposes.
Step-by-Step Solution

Partitioning the Global Catalog

Local vs Remote: The global catalog must be partitioned into local (China) and remote (global) datasets.
Metadata Management: Metadata about each dataset, such as schema, lineage, and usage patterns, should be managed separately for local and global resources.

Lineage Tracking

Centralized Lineage Storage: Maintain a centralized line of custody for lineage metadata to ensure consistency across regions.
Event-Driven Approach: Use event-driven architectures where changes in the data catalog trigger updates in the lineage store, ensuring real-time visibility into data transformations and movements.

Metrics Layer Design

Regional Metrics Aggregation: Implement a metrics layer that aggregates performance metrics at regional levels before sending them to a central repository.
Latency Optimization: Optimize data processing pipelines to minimize latency for local users while maintaining global performance standards.

#### Implementation
Details
Data Ingestion: Use secure transfer protocols (e.g., SFTP) or encrypted APIs to ingest data into the local cloud.
Data Processing: Leverage serverless computing and batch processing services in the local cloud to handle real-time and scheduled jobs.
Monitoring and Alerting: Implement robust monitoring and alerting mechanisms for both local and global systems.

#### Closing Summary

The design requires a nuanced approach that respects regulatory constraints while maintaining operational efficiency. By carefully partitioning the architecture, we can ensure compliance with Chinese data locality rules without sacrificing access or performance.

#### Common Mistakes/Follow-Ups

Overcomplicating Local vs Remote Databases: It's easy to over-engineer separate databases for local and remote data when a hybrid approach might suffice.
Neglecting Performance Impact: Failing to account for the potential increase in latency and cost due to regional processing could undermine the system’s effectiveness.

**Comparison:**

| Region | Data Latency (ms) |
|--------|------------------|
| China  | 50               |
| US-East-1 | 30              |
| EU-West-2 | 45              |

**Code:**

```python
# Example Python Code for Event-Driven Lineage Management
def update_lineage(event, context):
    # Update lineage metadata based on data changes
    pass

# Example of a Metrics Aggregation Function
def aggregate_metrics(region_data):
    aggregated = {}
    for region in region_data:
        for metric, value in region.get('metrics', {}).items():
            if metric not in aggregated:
                aggregated[metric] = 0
            aggregated[metric] += value
    return aggregated
```

**Example input / output:**

```text
# Input Data for Metrics Aggregation

Input:
{
  "globalCatalog": [
    { "id": 1, "region": "China" },
    { "id": 2, "region": "US-East-1" },
    { "id": 3, "region": "EU-West-2" }
  ],
  "metricsLayer": {
    "China": {"dataLatency": 50},
    "US-East-1": {"dataLatency": 30},
    "EU-West-2": {"dataLatency": 45}
  }
}

# Output Aggregated Metrics

Output:
{
  "globalMetrics": {
    "totalDataLatency": 38.67
  }
}
```

**Data:**

```json
{
  "globalCatalog": {
    "localCloud": [
      { "id": 1, "name": "LocalDatasetA", "region": "China" },
      { "id": 2, "name": "RemoteDatasetB", "region": "Global" }
    ],
    "remoteCloud": [
      { "id": 3, "name": "RemoteDatasetC", "region": "US-East-1" },
      { "id": 4, "name": "RemoteDatasetD", "region": "EU-West-2" }
    ]
  },
  "lineageStore": {
    "localCloud": [
      { "datasetId": 1, "lastUpdated": "2023-10-01T15:30Z", "transformations": ["TransformationX"] },
      { "datasetId": 2, "lastUpdated": "2023-10-02T09:45Z", "transformations": ["TransformationY"] }
    ],
    "remoteCloud": [
      { "datasetId": 3, "lastUpdated": "2023-10-01T16:00Z", "transformations": [] },
      { "datasetId": 4, "lastUpdated": "2023-10-02T10:00Z", "transformations": [] }
    ]
  },
  "metricsLayer": {
    "localCloud": {
      "dataLatency": 50,
      "throughput": 100
    },
    "remoteCloud": {
      "dataLatency": 30,
      "throughput": 80
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Is Local Data?
5. Store in Local Cloud
6. Secure Transfer to RemoteCloud
7. Process Data
8. Metadata Update
9. Metric Collection
10. Aggregate Metrics

