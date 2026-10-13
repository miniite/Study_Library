---
type: "practice"
difficulty: "Expert"
---

# You are setting a five-year event-streaming vision for an org running Kafka RabbitMQ and SQS in different pockets. How do you decide whether to consolidate onto one streaming substrate and how do you sequence that convergence without stalling product teams?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Kafka and the Lakehouse](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_and_the_Lakehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set a five-year event-streaming vision for an organization running Kafka, RabbitMQ, and SQS in different pockets, we need to assess whether consolidating these systems onto a single streaming substrate is feasible and beneficial. This involves understanding the current landscape, evaluating the technical, operational, and strategic implications, and creating a phased approach that minimizes disruption. Here’s how I would go about making this decision:

#### Triaging the Question

The core of the question revolves around determining whether to consolidate three different message brokers (Kafka, RabbitMQ, and SQS) onto one platform, ideally Kafka, given its widespread adoption in the industry. The key areas we need to explore include:

Technical Feasibility: Can these systems interoperate effectively?
Operational Viability: Will consolidation simplify operations and reduce costs?
Risk Management: How do we minimize disruptions during the transition without stalling product teams?
Long-term Strategy: What are the strategic benefits of standardizing on a single platform?

#### Assumptions
The organization has existing investments in Kafka, RabbitMQ, and SQS.
There are multiple product teams using these systems with varying levels of dependency.
Cost savings and operational efficiency are significant factors.

#### First-Principles Solution
Step 1: Evaluate Current Systems

First, conduct a detailed assessment of each system’s current state:

Kafka: Known for high throughput, partitioning, and distributed architecture; popular in large-scale data processing.
RabbitMQ: Good for message queuing with features like retries, dead-letter exchanges, and robustness.
SQS: Simple queue service for non-persistent messaging within AWS.
Step 2: Interoperability Analysis

Evaluate the potential to interoperate between these systems:

Kafka & RabbitMQ: Kafka Connect can be used to stream data from RabbitMQ topics. However, this requires significant integration effort.
Kafka & SQS: AWS has native integrations that can simplify the transition for existing SQS users.
Step 3: Operational Impact

#### Cons
ider the operational implications:

Data Consistency: Ensure seamless data flow and consistency across systems.
Monitoring and Logging: Standardize monitoring practices to reduce overhead and improve visibility.
Security: Implement consistent security measures across all services.
Step 4: Phased Approach for Convergence

To minimize disruptions, we can adopt a phased approach:

Pilot Project: Begin with a small pilot project to validate technical feasibility and operational impact.
Phased Rollout: Gradually migrate critical components starting from less complex ones.
Training and Documentation: Provide comprehensive training and detailed documentation for product teams.
Continuous Improvement: Regularly review the system’s performance and make adjustments as needed.

#### Closing Summary

#### Cons
olidating onto a single streaming substrate, such as Kafka, can significantly enhance operational efficiency and reduce costs while maintaining or improving service quality. By adopting a phased approach, we can manage the transition smoothly without stalling product teams. Key considerations include evaluating technical feasibility, ensuring interoperability, managing risks, and providing necessary support.

#### Common Mistakes/Follow-Ups
Overlooking Interoperability Issues: Failing to account for the complexities in integrating different systems.
Ignoring Operational Risks: Not properly planning for data consistency, monitoring, and security.
Disrupting Product Teams: Underestimating the impact on existing projects and failing to provide adequate support.

**Comparison:**

```
System	Pros	Cons
Kafka	High throughput, distributed	Steeper learning curve, complex setup
RabbitMQ	Robust messaging, retries	Limited scalability for large volumes
SQS	Simple, non-persistent messaging	AWS-specific, limited features
```

**Code:**

```python
# Example of Kafka Connect configuration for RabbitMQ source
{
    "name": "rabbitmq-source",
    "config": {
        "connector.class": "io.confluent.connect.rabbitmq.RabbitMQSourceConnector",
        "topics": "queue_name",
        "confluent.topic.bootstrap.servers": "kafka_broker_addresses",
        "confluent.topic.replication.factor": 3,
        "confluent.username": "rabbitmq_user",
        "confluent.password": "rabbitmq_password"
    }
}
```

**Example input / output:**

```text
Input:
- Existing Kafka, RabbitMQ, and SQS systems in different pockets.
Output:
- A phased approach for consolidating onto a single Kafka platform.
Steps:
1. Pilot project
2. Phased rollout
3. Training and documentation
4. Continuous improvement
```

**Data:**

```json
{
    "topic": "example_topic",
    "partitions": [
        {"partition": 0, "replicas": [1]},
        {"partition": 1, "replicas": [2]}
    ]
}
```

**Flow:**

1. Yes
2. Start
3. Evaluate Systems
4. Pilot Project
5. Phased Rollout
6. Training and Documentation
7. Continuous Improvement

