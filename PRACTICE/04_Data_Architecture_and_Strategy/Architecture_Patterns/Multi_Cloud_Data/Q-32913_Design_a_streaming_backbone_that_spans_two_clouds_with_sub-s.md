---
type: "practice"
difficulty: "Expert"
---

# Design a streaming backbone that spans two clouds with sub-second replication for a fraud-detection use case — what consistency guarantees do you sacrifice and how do you communicate that to risk teams?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-cloud Portability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Multi-cloud_Portability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In designing a multi-cloud streaming backbone, we need to strike a balance between achieving high availability and ensuring data consistency across two clouds. The primary trade-off involves choosing an eventual consistency model to achieve sub-second replication, which will require careful communication with risk teams about the associated risks.

#### Triaging the Question

The core of this question revolves around designing a robust streaming architecture that spans two cloud environments while maintaining sub-second data replication for a critical fraud-detection use case. The key aspect is understanding and managing consistency guarantees to ensure both system reliability and compliance with regulatory requirements.

#### Assumptions
Latency Requirements: Sub-second latency is crucial for real-time fraud detection.
Fault Tolerance: Both clouds must be highly available, with minimal downtime.
Regulatory Compliance: The solution needs to adhere to relevant data protection regulations (e.g., GDPR).
Risk Management: Risk teams need clear communication about the chosen architecture and its implications.

#### First-Principles Solution
1. Choose a Consistency Model: Eventual Consistency

To achieve sub-second replication across two clouds, we must adopt an eventual consistency model. This means that while data will be available near-instantly in both environments, it may take a short period for all copies to converge.

2. Design the Data Flow
Event Generation: Use a message broker (e.g., Apache Kafka) to capture fraud detection events as they occur.
Replication Layer: Implement a replication layer that ensures data is propagated across clouds within microseconds.
Processing Layer: Deploy processing pipelines in both clouds to handle and analyze events simultaneously.
3. Ensure High Availability
Multi-AZ Deployment: Use multi-active availability zones (AZs) or regions for each cloud provider to minimize downtime.
Redundant Infrastructure: Set up redundant networks, storage, and compute resources to ensure high availability.
Failover Mechanism: Implement failover mechanisms to redirect traffic in case of a single-cloud failure.
4. Manage Consistency Guarantees

To maintain compliance with strict regulatory requirements, we must implement a mechanism that periodically ensures data consistency across both clouds. This could involve:

Periodic Reconciliation: Use periodic reconciliation processes to ensure all copies are eventually consistent.
Conflict Resolution: Implement conflict resolution strategies to handle edge cases where data discrepancies arise.
Communicating the Solution
1. Explain the Consistency Model

To risk teams, explain that while sub-second replication is achieved, the system operates under an eventual consistency model due to the nature of cross-cloud communication delays and network constraints.

2. Highlight Risks
Data Latency: There may be a small window where data inconsistencies exist.
Risk Mitigation: Emphasize the robust failover mechanisms and periodic reconciliation processes in place to minimize risk.
Regulatory Compliance: Ensure that all steps are taken to meet regulatory standards, even if it means sacrificing some real-time consistency.
3. Provide Assurances
Transparency: Be transparent about how data is handled and why eventual consistency is necessary.
Compliance Checks: Regularly perform compliance checks to ensure adherence to regulatory requirements.

#### Closing Summary

In summary, designing a multi-cloud streaming backbone for fraud detection involves adopting an eventual consistency model to achieve sub-second replication. While this sacrifices some real-time consistency guarantees, it ensures high availability and meets critical latency requirements. Clear communication with risk teams about the chosen architecture and its implications is crucial to manage any potential risks.

#### Common Mistakes/Follow-Ups
Overlooking Data Latency: Ignoring the inherent delays in cross-cloud communication can lead to false assumptions about real-time consistency.
Neglecting Compliance: Failing to implement periodic reconciliation and conflict resolution processes can result in non-compliance with regulatory standards.
Underestimating Risk Management: Not adequately communicating the risk trade-offs may lead to unrealistic expectations from stakeholders.

**Comparison:**

```
Event Type	Cloud 1	Cloud 2
Fraud Detection	Yes (0.5ms)	Yes (0.8ms)
Reconciliation Status	In Progress	In Progress
```

**Code:**

```python
# Example of a simple Kafka producer configuration for event generation
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['cloud1-bootstrap:9092', 'cloud2-bootstrap:9092'])

event = {"fraudDetection": True, "amount": 500}
future = producer.send('frauddetection-topic', b'{"fraudDetection": true, "amount": 500}')
result = future.get(timeout=60)
```

**Example input / output:**

```

Input: Fraud detection event generated in cloud 1.

JSON
Copy
{
  "event": {
    "timestamp": "2023-10-01T14:00:00Z",
    "fraudDetection": true,
    "amount": 500
  }
}

Output: Event replicated to both cloud 1 and cloud 2 within microseconds.

JSON
Copy
{
  "event": {
    "timestamp": "2023-10-01T14:00:00Z",
    "fraudDetection": true,
    "amount": 500
  }
}
```

**Data:**

```json
{
  "event": {
    "timestamp": "2023-10-01T14:00:00Z",
    "fraudDetection": true,
    "amount": 500
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Event Generation
6. Is event in Cloud 1?
7. Replicate to Cloud 2
8. Is event in Cloud 2?
9. Periodic Reconciliation
10. Processing Pipeline
11. Analyze Events

