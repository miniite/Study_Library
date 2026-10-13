---
type: "practice"
difficulty: "Deep"
---

# You're asked whether to standardize the org on a single broker or run purpose-fit brokers per workload (log for streaming, queue for work-distribution, pub/sub for notifications). Defend a position and describe the operational tax of the polyglot choice.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fat vs Thin Events](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address whether it's better to standardize our organization on a single message broker or run purpose-fit brokers per workload, we need to consider several factors including technical feasibility, performance, operational complexity, and organizational culture. The key decision points are around the trade-offs between centralization and specialization.

#### Triaging the Question

The question is essentially about choosing between a monolithic (single) vs polyglot (multiple specialized) approach for message brokers in our organization. The interviewer is probing your understanding of the architecture principles, scalability considerations, operational overheads, and how these choices impact overall system reliability and maintainability.

#### Assumptions
and First Principles
Current System: We are evaluating a migration or enhancement to our current messaging infrastructure.
Workloads: Our organization handles various types of workloads such as logging, task scheduling, and notifications.
Polyglot Approach: Running purpose-fit brokers like Kafka for logs, RabbitMQ for queues, and Nats for pub/sub.
Single Broker: Standardizing on a single broker that can handle all types of messaging patterns.
Step-by-Step Solution
Monolithic (Single Broker) Approach
Simplicity and Consistency:
Easier to manage a single technology stack.
Centralized governance, policies, and security.
Scalability Concerns:
One broker may not efficiently handle all types of workloads.
Potential bottleneck or performance degradation when scaling up.
Polyglot (Multiple Brokers) Approach
Specialization and Optimization:
Each workload has its optimal configuration for performance, reliability, and ease of management.
Operational Tax:
Increased complexity in managing multiple technologies.
Potential interoperability issues between brokers.
Cost and Maintenance:
Higher initial setup and ongoing maintenance costs.
Evaluating the Polyglot Approach

Let's delve into why a polyglot approach might be more suitable for our organization:

Performance Optimization:

Kafka excels in log aggregation and real-time data processing.
RabbitMQ is ideal for task distribution with its queueing capabilities.
Nats handles pub/sub patterns efficiently, ensuring low-latency message delivery.

Operational Complexity:

Each broker has unique management tools and operational practices.
However, this complexity can be mitigated by leveraging automation and DevOps best practices.

Scalability and Reliability:

Kafka clusters are optimized for high-throughput data ingestion and storage.
RabbitMQ’s robust queuing system ensures reliable task distribution.
Nats' pub/sub model is lightweight, making it suitable for event-driven architectures.

Security Considerations:

Different brokers might have varying security features tailored to specific use cases.
Implementing consistent security policies across multiple brokers requires careful planning but can be effective.

Maintenance and Troubleshooting:

Having dedicated teams or roles responsible for each broker simplifies maintenance.
Clear documentation and standard operating procedures help in troubleshooting and scaling.

#### Closing Summary

In conclusion, while a monolithic approach offers simplicity and easier management, the polyglot approach of running purpose-fit brokers per workload is often more efficient and scalable. The key benefit lies in optimizing performance for specific use cases without compromising on reliability or security. However, this comes with an increased operational tax due to higher complexity.

#### Common Mistakes
and Follow-Ups
Failing to Consider Performance Impacts: Sometimes organizations choose a single broker solely based on ease of management without assessing the impact on performance.
Ignoring Interoperability Issues: Running multiple brokers can lead to challenges in ensuring seamless message delivery across different systems.
Overlooking Maintenance Overhead: Multiple brokers require more resources for setup, monitoring, and maintenance.

#### Follow-up
questions might include:

How would you handle data transfer between different brokers?
What are the key metrics to monitor for each type of broker?
Can you describe a real-world scenario where polyglot messaging helped in your past projects?

**Comparison:**

| Broker   | Throughput (MB/s) | Latency (ms) | Scalability |
|----------|------------------|-------------|------------|
| Kafka    | 100              | 5           | High       |
| RabbitMQ | 20               | 10          | Medium     |
| Nats     | 50               | 3           | High       |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Single Broker] --> B1[Centralized Management]
    A --> B2[Simplified Governance]
    A --> B3[Standard Security Policies]

    C[Polyglot Brokers] --> D1[Specialization for Performance]
    C --> D2[Different Operations Teams]
    C --> D3[Interoperability Challenges]

    E[Workload A: Logging] --> F[Kafka]
    E --> G[RabbitMQ]
    E --> H[Nats]

    F --> I[High-Throughput Ingestion]
    G --> J[Task Distribution Queues]
    H --> K[Pub/Sub Events]
```

**Example input / output:**

```text
| Input | Output                    |
|-------|---------------------------|
| Log   | Produced by Kafka         |
| Task  | Enqueued in RabbitMQ      |
| Event | Published via Nats        |

Example:
- Input: "INFO - Application Started"
- Output: Message sent to Kafka topic `application_logs`
```

**Data:**

```yaml
workloads:
  - name: Logging
    broker: Kafka

  - name: Task Scheduling
    broker: RabbitMQ

  - name: Notifications
    broker: Nats
```

**Flow:**

1. Application
2. Kafka
3. Log Aggregation Service
4. Data Storage
5. Application
6. RabbitMQ
7. Task Scheduling System
8. Worker Nodes
9. Notification Service
10. Nats
11. User Interface

