---
type: "practice"
difficulty: "Expert"
---

# How would you design topics for a multi-tenant system: topic-per-tenant vs key-per-tenant, and the trade-offs?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topic Design for Re-sync](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Topic_Design_for_Re-sync.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing Kafka topics for a multi-tenant system, we need to balance tenant isolation with efficient resource utilization. The two primary strategies are "topic-per-tenant" and "key-per-tenant." I'll explain each approach, their trade-offs, and the considerations that make one more suitable than the other depending on your specific needs.

TL;DR
Topic-per-Tenant: Each tenant has its own topic, ensuring complete isolation but increasing complexity in management.
Key-per-Tenant: Uses a single topic with a key-based partitioning strategy to maintain isolation while reducing topic count and making management easier.
The choice depends on the data sensitivity, number of tenants, and required performance characteristics.

#### Triage

The question is asking us to compare two strategies for handling multi-tenancy in Kafka: one where each tenant has its own topic (topic-per-tenant) versus one that uses a single topic with key-based partitioning. The interviewer wants to understand the trade-offs between isolation, resource usage, and management complexity.

#### First Principles Solution
Topic-per-Tenant:
Isolation: Perfect isolation since each tenant has its own topic.
Scalability: Each tenant can scale independently without affecting others.
Management Complexity: Higher management overhead due to more topics to manage and monitor.
Key-per-Tenant:
Isolation: Achieves strong isolation by using a key-based partitioning strategy where keys correspond to tenants, ensuring data from different tenants does not mix in the same partitions.
Scalability: Easier to scale since fewer topics exist.
Management Complexity: Lower management overhead as you only need to manage and monitor one topic.
Detailed Analysis
1. Isolation vs Scalability
Topic-per-Tenant: Each tenant has its own topic, ensuring that even if a single topic fails, it does not affect other tenants. However, this approach requires careful configuration of replication factors and retention policies for each topic.
Key-per-Tenant: While the data is still isolated within the same topic, this method relies on key-based partitioning to ensure that data from different tenants does not mix. This can be more complex to set up but reduces the number of topics needed.
2. Management Complexity
Topic-per-Tenant: Managing multiple topics can become cumbersome as the number of tenants grows. You need to configure and monitor each topic separately.
Key-per-Tenant: With a single topic, you only manage one Kafka topic configuration, which simplifies monitoring and management.
3. Data Sensitivity
If data sensitivity is high, you might prefer isolation through topic-per-tenant because it ensures no data mixing.
For less sensitive data or scenarios where the benefit of reduced management overhead outweighs the risk of data mixing, key-per-tenant could be more appropriate.
Best Practices and Considerations
Tenant Size: If there are a small number of tenants, topic-per-tenant might be simpler to manage.
Data Volume: High volume per tenant might require separate topics for better performance and resource allocation.
Performance Requirements: Key-per-tenant can be more efficient in terms of resource usage but requires careful tuning of partitioning strategies.

#### Closing Summary

The choice between topic-per-tenant and key-per-tenant depends on the specific requirements of your multi-tenant system, including data sensitivity, performance needs, management complexity, and the number of tenants. Topic-per-tenant provides perfect isolation at the cost of increased management overhead, while key-per-tenant achieves similar isolation with a single topic but requires careful partitioning.

#### Common Mistakes/Follow-Ups
Not Considering Key-based Partitioning: Failing to recognize that key-based partitioning can achieve strong isolation without the need for multiple topics.
Overlooking Management Complexity: Ignoring the overhead of managing many Kafka topics, which can become a significant burden in large-scale systems.

FIELD:code_blocks>>>

PYTHON
# Example Python code to create Kafka topics using topic-per-tenant approach

from kafka.admin import KafkaAdminClient, NewTopic

def create_topics(topics):
    admin_client = KafkaAdminClient(bootstrap_servers="localhost:9092")
    new_topics = [NewTopic(topic, num_partitions=1, replication_factor=3) for topic in topics]
    admin_client.create_topics(new_topics)

FIELD:data_blocks>>>

JSON
{
  "topics": ["tenant-1-topic", "tenant-2-topic"],
  "num_partitions": 1,
  "replication_factor": 3
}

FIELD:input_output>>> Input:

PYTHON
create_topics(["tenant-1-topic", "tenant-2-topic"])

Output:

PLAINTEXT
Topics created successfully.

FIELD:tables>>> | Strategy | Isolation | Scalability | Management Complexity | |----------|-----------|-------------|----------------------| | Topic-per-Tenant | Perfect | Low | High | | Key-per-Tenant | Strong | High | Low |

FIELD:flowcharts>>>

Topic-per-Tenant
Isolation
Perfect
Scalability
Low
Management Complexity
High
Key-per-Tenant
Isolation
Strong
Scalability
High
Management Complexity
Low

FIELD:self_check>>> Confirmed correctness and completeness.

