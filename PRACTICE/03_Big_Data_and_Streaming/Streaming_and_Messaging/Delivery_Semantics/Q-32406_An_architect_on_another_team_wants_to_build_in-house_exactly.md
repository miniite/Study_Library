---
type: "practice"
difficulty: "Deep"
---

# An architect on another team wants to build in-house exactly-once middleware. Run the build-vs-buy analysis against Kafka transactions, Pulsar, and cloud-native queues, weighting long-term maintenance and hiring risk.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Exactly Once](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Cost_of_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To run a build-vs-buy analysis on exactly-once middleware, we need to evaluate multiple factors including implementation complexity, reliability, performance, scalability, supportability, and cost. The key comparison points are Kafka transactions, Pulsar, and cloud-native queues like Amazon SQS or Google Pub/Sub. Here’s a detailed breakdown of each:

First, let's establish the criteria:

#### Implementation
Complexity: How challenging is it to implement exactly-once semantics?
Reliability and Consistency: How well does the system ensure message delivery?
Performance: What impact will it have on processing latency and throughput?
Scalability: Can it handle large volumes of messages without degradation?
Supportability: Is there a robust ecosystem, community, and support for maintenance?

#### Triaging the Question

The core question is to weigh the long-term benefits of building in-house against buying established solutions (Kafka transactions, Pulsar, or cloud-native queues). We need to consider not just immediate cost but also future costs like maintenance, hiring risk, and potential operational challenges.

#### First-Principles Solution
Kafka Transactions

Complexity: Implementing exactly-once semantics in Kafka requires deep understanding of its transactional guarantees. You'd need to manage transactions manually, which can be error-prone. Reliability: Kafka’s built-in transaction support is reliable but requires careful configuration and handling of retries. Performance: Adding transactional writes can introduce additional latency. However, this can be mitigated with proper tuning. Scalability: Highly scalable; works well in distributed systems. Supportability: Strong community and enterprise support, but implementing custom solutions may require significant expertise.

Pulsar

Complexity: Pulsar natively supports exactly-once semantics through its transactional write feature. This reduces the implementation complexity compared to Kafka. Reliability: Pulsar ensures message reliability through its transactional guarantees, offering strong consistency. Performance: Performance overhead is lower than Kafka due to its optimized architecture for transactions. Scalability: Very scalable; designed with high availability and fault tolerance in mind. Supportability: Growing community support; Apache Pulsar provides robust documentation.

Cloud-Native Queues (e.g., SQS, Pub/Sub)

Complexity: These services abstract away many complexities. Implementing exactly-once semantics is simpler as the service handles retries and deduplication. Reliability: Highly reliable with built-in features like message deduplication and at-least-once delivery; ensuring exactly-once requires additional logic. Performance: Lower overhead compared to self-managed solutions due to optimized services. However, latency might be higher for complex use cases. Scalability: Cloud-native offerings typically scale well but may have limitations based on provider quotas. Supportability: Robust support from cloud providers with extensive documentation and community.

#### Closing Summary

Given the analysis above, Pulsar seems like a balanced choice due to its native exactly-once semantics and strong reliability. Cloud-native queues can be simpler to implement if leveraging existing services is sufficient for your needs. Building in-house requires significant expertise but offers flexibility and potentially lower long-term maintenance costs.

#### Common Mistakes/Follow-ups
Overlooking Long-Term Maintenance: Failing to account for the cost of maintaining custom solutions.
Ignoring Community Support: Underestimating the value of robust community support, especially when issues arise.
Simplifying Reliability Assumptions: Assuming exactly-once semantics are automatically provided by cloud services without additional implementation effort.

**Comparison:**

```
Service	Complexity	Reliability	Performance Overhead	Scalability	Supportability
Kafka Transactions	High (5)	Good (8)	High (7)	High (9)	Strong (9)
Pulsar	Medium (3)	Very Good (9)	Low (6)	High (9)	Growing (8)
Cloud-Native Queues (SQS)	Low (2)	Good (7)	Low (5)	Moderate (7)	Strong (10)
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Build In-House] --> B1[High Complexity];
    A --> B2[Long-Term Maintenance];
    A --> B3[Hiring Risk];

    C[Buy Kafka Transactions] --> D1[Complex Implementation];
    C --> D2[Good Reliability];
    C --> D3[Higher Performance Overhead];

    E[Buy Pulsar] --> F1[Simplified Exactly-Once Semantics];
    E --> F2[Strong Community Support];
    E --> F3[Reliable but with Some Latency];

    G[Buy Cloud-Native Queues] --> H1[Easier to Implement];
    G --> H2[Lower Long-Term Maintenance];
    G --> H3[Higher Operational Risk];
```

**Example input / output:**

```text
Input:
- Message Delivery Semantics Analysis Requested
Output:
- Kafka Transactions: Complexity 5/10, Reliability 8/10, Performance Overhead 7/10, Scalability 9/10, Supportability 9/10
- Pulsar: Complexity 3/10, Reliability 9/10, Performance Overhead 6/10, Scalability 9/10, Supportability 8/10
- Cloud-Native Queues (SQS): Complexity 2/10, Reliability 7/10, Performance Overhead 5/10, Scalability 7/10, Supportability 10/10
```

**Data:**

```yaml
services:
  - name: Kafka Transactions
    complexity: high
    reliability: good
    performance_overhead: high
    scalability: high
    supportability: strong

  - name: Pulsar
    complexity: medium
    reliability: very_good
    performance_overhead: low
    scalability: high
    supportability: growing

  - name: SQS
    complexity: low
    reliability: good
    performance_overhead: low
    scalability: moderate
    supportability: strong
```

**Flow:**

1. Build In-House
2. High Complexity
3. Long-Term Maintenance
4. Hiring Risk
5. Kafka Transactions
6. Complex Implementation
7. Good Reliability
8. Higher Performance Overhead
9. Pulsar
10. Simplified Exactly-OnceSemantics
11. Strong Community Support
12. Reliable with Some Latency
13. Cloud-Native Queues (SQS)
14. Easier to Implement
15. Lower Long-Term Maintenance
16. Higher Operational Risk

