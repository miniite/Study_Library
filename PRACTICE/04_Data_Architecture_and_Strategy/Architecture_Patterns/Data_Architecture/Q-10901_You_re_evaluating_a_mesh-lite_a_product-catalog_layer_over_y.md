---
type: "practice"
difficulty: "Expert"
---

# You're evaluating a 'mesh-lite': a product-catalog layer over your existing central observability platform so teams own their telemetry data products but the platform stays central. Is this a pragmatic hybrid or worst-of-both, and where does it break down at scale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Mesh Observability Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Centralized_vs_Mesh_Observability_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In evaluating a 'mesh-lite' architecture, we need to consider whether it offers practical benefits that outweigh its drawbacks. The core idea is to introduce a product-catalog layer over an existing central observability platform, allowing teams to own their telemetry data products while retaining centralized management and control. This approach aims to balance the flexibility of distributed ownership with the efficiency and consistency provided by a unified observability platform.

The key question here is: Can this hybrid model maintain scalability, reliability, and governance at scale without compromising on performance or ease of use? We need to consider several aspects:

Ownership and Data Consistency: How will teams ensure data consistency when they own their telemetry products?
Performance and Scalability: Will the 'mesh-lite' approach introduce bottlenecks as more teams adopt it?
Governance and Compliance: How can we enforce governance policies across distributed ownership models?

Let's break this down step by step.

Ownership and Data Consistency

In a 'mesh-lite' model, each team has its own data product but still relies on the central observability platform for processing and storage. The challenge lies in ensuring that these teams maintain consistent practices without disrupting the overall system. A potential solution could be to implement standardized schemas and interfaces, enforce policies via API gateways, or use a common metadata store.

Performance and Scalability

The performance and scalability of this hybrid model depend heavily on how effectively we can scale out the central observability platform while also scaling in with individual team products. Key considerations include:

Data Aggregation: How will data be aggregated from multiple sources without introducing significant latency?
Query Latency: Can distributed queries maintain low latency, or do they become a bottleneck as more teams join?
Governance and Compliance

Governance is critical to ensure that all teams follow best practices. We need mechanisms for:

Data Security: Enforcing access controls and encryption.
Compliance: Ensuring data compliance with internal and external regulations.
Full Solution: First Principles

To evaluate this hybrid model, we must first define the core principles it should adhere to:

Centralized Control: Maintain centralized control over critical operations such as data processing pipelines, metadata management, and security policies.
Decentralized Ownership: Allow teams to own their telemetry products while ensuring they follow a set of predefined standards and practices.
Scalability: Ensure the system can scale horizontally and vertically without significant degradation in performance.

#### Implementation
Central Observability Platform: This platform will handle data collection, processing, storage, and analysis. It should have robust capabilities to support high-volume telemetry data.
Product-Catalog Layer: A layer that allows teams to define and manage their own telemetry products using standardized interfaces and schemas. This can include APIs for collecting, storing, and querying data.
Governance Tools: Implement tools such as API gateways, policy engines, and metadata stores to enforce consistency and security.

#### Common Mistakes
Ignoring Central Governance: Failing to establish clear governance policies can lead to inconsistent practices and potential security vulnerabilities.
Over-Engineering Decentralized Ownership: Without proper guidelines, decentralized ownership might result in suboptimal data quality and performance issues.
Neglecting Scalability Considerations: Poorly designed scaling mechanisms can introduce bottlenecks as the system grows.

#### Closing Summary

A 'mesh-lite' architecture has the potential to be a pragmatic hybrid solution by leveraging the strengths of both centralized control and decentralized ownership. However, careful consideration is needed to ensure that scalability, performance, and governance are maintained at scale. By defining clear principles and implementing robust tools, we can mitigate many of the risks associated with this approach.

#### Follow-Ups
Performance Monitoring: How will you monitor query latency and data aggregation performance?
Security Audits: What mechanisms will be in place to ensure ongoing security audits across teams?

**Comparison:**

| Team | Timestamp   | Metrics |
|------|-------------|---------|
| T12345| 1687509000  | [12, 15, 18] |
| T12345| 1687509002  | [14, 17, 20] |
| T67890| 1687509001  | [13, 14, 16] |

**Code:**

```python
# Example code for defining a standardized schema using JSON Schema
{
    "$schema": "http://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
        "teamID": {"type": "string"},
        "timestamp": {"type": "integer"},
        "metrics": {"type": "array", "items": {"type": "number"}}
    },
    "required": ["teamID", "timestamp", "metrics"]
}
```

**Example input / output:**

```text
# Example input and output for a data aggregation query

Input:
- Team A sends telemetry data at timestamps 1687509000 and 1687509002.
- Team B sends telemetry data at timestamp 1687509001.

Output:
- Aggregated data for all teams, ensuring no duplicates or missing values.
```

**Data:**

```yaml
# Sample data for a telemetry product definition
product:
  - teamID: T12345
    timestamp: 1687509000
    metrics: [12, 15, 18]
  - teamID: T67890
    timestamp: 1687509001
    metrics: [13, 14, 16]
```

**Flow:**

1. Telemetry Data Collection
2. Central Observability Platform
3. Product-Catalog Layer
4. API Gateway
5. Governance Policy Engine
6. Metadata Store
7. Decentralized Ownership
8. Real-Time Aggregation
9. Performance Monitoring

