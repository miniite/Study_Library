---
type: "practice"
difficulty: "Deep"
---

# You're setting the org standard for how much aggregation happens at the edge collector versus in the central stream. What's the trade-off between edge pre-aggregation (cheap, lossy) and central raw ingestion (expensive, flexible), and how do you let teams tune it safely?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Processing Time vs Event Time](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Processing_Time_vs_Event_Time.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're setting standards for data aggregation in a real-time streaming architecture, deciding between edge pre-aggregation at the collector or central raw ingestion. The trade-off revolves around cost, flexibility, and the impact on system performance and accuracy. We need to balance these factors while allowing teams to tune their configurations safely.

The key is to understand that edge pre-aggregation can reduce bandwidth and storage costs but may introduce lossiness in data quality. Conversely, central raw ingestion ensures data integrity at the cost of higher processing demands and storage. By providing a flexible tuning mechanism, we can adapt to varying use cases and maintain both cost-effectiveness and data fidelity.

First-principles reasoning starts with defining our goals: reduce latency, manage costs, ensure data accuracy, and enable scalability. Edge pre-aggregation works well for scenarios where the volume is high and bandwidth constraints are critical. Central raw ingestion suits environments that prioritize data integrity and can afford higher processing overheads. Balancing these requires a nuanced approach, leveraging both edge and central strategies.

Let's break down the components:

Edge Pre-Aggregation: This involves aggregating data at the edge collector to reduce volume before transmitting it to the central system. It’s cost-effective due to reduced bandwidth and storage needs but may lead to loss of finer granularity.
Central Raw Ingestion: Here, data is transmitted in its raw form to a central stream where aggregation can occur post-transmission. This approach ensures no data loss at the expense of higher processing costs.

To implement this safely, we should:

Define clear policies and thresholds for when edge pre-aggregation kicks in versus central ingestion.
Provide tools or interfaces that allow teams to adjust these settings based on their use cases.
Implement monitoring and alerting mechanisms to ensure the system performs as expected under different configurations.

For example, if a team needs high accuracy but has bandwidth constraints, they might set up edge aggregation with lower data retention periods. Conversely, for less critical applications where flexibility is key, central raw ingestion can be used.

In summary, we need to design a hybrid approach that allows flexible tuning while ensuring cost and data integrity. The architecture should support both modes, providing teams the ability to choose based on their specific requirements.

**Comparison:**

```
Scenario	Edge Pre-Aggregation	Central Raw Ingestion
Cost	Low	High
Data Loss	Yes	No
Flexibility	Limited	High
Scalability	Medium	High
Bandwidth Usage	Reduced	Unchanged
```

**Code:**

```python
# Example configuration settings for edge pre-aggregation vs central raw ingestion

class DataIngestionConfig:
    def __init__(self, max_bandwidth_gb, data_retention_hours):
        self.max_bandwidth_gb = max_bandwidth_gb
        self.data_retention_hours = data_retention_hours

    def is_edge_pre_aggregation_needed(self, current_bandwidth_usage, threshold=80):
        return current_bandwidth_usage > (threshold * self.max_bandwidth_gb)

# Example usage
config = DataIngestionConfig(max_bandwidth_gb=10.0, data_retention_hours=24)
current_usage = 9.5  # GB

if config.is_edge_pre_aggregation_needed(current_usage):
    print("Edge pre-aggregation is recommended.")
else:
    print("Central raw ingestion is preferred.")
```

**Example input / output:**

```

Given the input:

max_bandwidth_gb = 10.0
data_retention_hours = 24
current_bandwidth_usage = 9.5

The output is:

PLAINTEXT
Copy
Edge pre-aggregation is recommended.
```

**Data:**

```json
{
  "max_bandwidth_gb": 10.0,
  "data_retention_hours": 24,
  "current_bandwidth_usage": 9.5
}
```

**Flow:**

1. Yes
2. No
3. Edge Pre-Aggregation
4. Is current bandwidth usage >threshold?
5. Apply pre-aggregation
6. Central Raw Ingestion

