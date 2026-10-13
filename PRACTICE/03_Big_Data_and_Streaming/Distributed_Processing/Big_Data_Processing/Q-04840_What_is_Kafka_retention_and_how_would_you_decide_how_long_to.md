---
type: "practice"
difficulty: "Basic"
---

# What is Kafka retention and how would you decide how long to keep events on a topic?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Kafka Retention and Storage](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Kafka retention refers to the mechanism that determines how long data stored in a Kafka topic remains available before being deleted by the system. It is crucial for ensuring data availability while also managing storage costs and resource utilization.

The core of this question is to understand your approach to deciding on an appropriate retention period, which involves balancing factors such as business requirements, storage costs, and operational constraints. Let's break it down step by step.

#### Triaging the Question
What is Kafka Retention? - This is about understanding how data is retained in a Kafka topic.
How to Decide on Retention Duration? - The interviewer is probing your thought process for making decisions based on real-world constraints and requirements.

#### First Principles Solution
Business Requirements: Determine what events are critical for business operations, compliance, or analysis. For example, if you need data for financial audits, it might be necessary to retain data indefinitely.
Storage Costs: Evaluate the cost of storing large volumes of data over long periods. High costs can lead to operational inefficiencies and potentially impact other critical services.
Operational Constraints: Consider the capacity limits of your Kafka cluster and how retaining too much data could affect performance and reliability.
Step-by-Step Solution

Identify Critical Business Needs:

Analyze use cases where data retention is crucial, such as regulatory compliance, historical analysis, or critical business operations.
For instance, if you are processing financial transactions, regulatory requirements might mandate retaining data for 7 years.

Assess Storage Costs:

Calculate the cost of storing data over different periods using your current Kafka setup.

#### Cons
ider alternative storage solutions like cheaper storage tiers or archiving strategies to reduce costs without compromising on critical data availability.

Evaluate Performance and Capacity:

Determine how much data can be stored before it impacts cluster performance, such as lag in processing new events due to old data.
Use monitoring tools to track metrics like retention period impact on Kafka's performance.

Define a Retention Policy:

Based on the above considerations, set an optimal retention period that balances cost and operational requirements.
For example, if regulatory compliance requires at least 5 years of data but storage costs are high, you might decide to retain data for 3-5 years with periodic archiving to a cheaper tier.

#### Closing Summary

Deciding on the Kafka retention period involves a careful balance between business needs, storage costs, and operational constraints. A pragmatic approach would be to first understand critical requirements, then assess cost implications, and finally define a policy that meets both current and future needs while being fiscally responsible.

#### Common Mistakes/Follow-Ups

Mistake: Failing to consider the impact on Kafka performance due to excessive data retention.

#### Follow-Up
Discussing potential solutions like tiered storage or periodic archiving strategies.

Mistake: Ignoring long-term regulatory compliance requirements, leading to legal issues.

#### Follow-Up
Highlighting the importance of understanding and aligning with industry regulations.

**Comparison:**

```
Use Case	Retention Period (Years)	Storage Tier
Financial Audit	7	Primary Storage
Customer Activity Logs	3	Secondary Storage
Cost Analysis	Cost Impact (USD)
High-Cost Data	$10,000/GB/year
Low-Cost Data	$2,500/GB/year
```

**Code:**

```python
# Example Python code to set Kafka retention policy (pseudo-code)
from kafka.admin import KafkaAdminClient, NewTopic

admin_client = KafkaAdminClient(bootstrap_servers='localhost:9092')
topic_configs = {'retention.ms': 604800000}  # 7 days in milliseconds
new_topic = NewTopic(name='example-topic', num_partitions=3, replication_factor=1, config=topic_configs)
admin_client.create_topics(new_topics=[new_topic])
```

**Example input / output:**

```text
Input:
- Use case: Financial audit (retention policy = 7 years)
- Cost analysis: Storage cost of $X/GB/month for long-term data

Output:
- Retention period: 3-5 years with periodic archiving to cheaper tier storage
- Archiving strategy: Monthly archival of non-critical data
```

**Data:**

```json
{
  "retention_policy": {
    "critical_business_needs": ["financial_audit", "compliance"],
    "storage_costs": {
      "high_cost_data": ["logistics_transactions", "customer_activity_logs"],
      "low_cost_data": ["non-critical_events"]
    },
    "performance_metrics": {
      "lag_in_ms": 100,
      "throughput_per_sec": 50
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Critical Business Needs
4. Regulatory Compliance
5. Set Retention Period
6. 3-5 Years with Archiving
7. 7+ Years for Compliance
8. Assess Storage Costs
9. High-Cost Data
10. Consider Tiered Storage
11. Primary + Secondary Tiers
12. Low-Cost Data
13. Simple Retention
14. Single Storage Tier

