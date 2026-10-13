---
type: "practice"
difficulty: "Deep"
---

# Argue whether the observability warehouse should ever be system-of-record for anything, or purely a derived copy of the event stream. What goes wrong when teams start treating a rolled-up telemetry table as authoritative for capacity or billing decisions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Mesh Observability Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Centralized_vs_Mesh_Observability_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this discussion, we need to decide whether an observability warehouse should be used as a system-of-record (SOR) for any critical data, or if it should only serve as a derived copy of an event stream. This decision involves considering the trade-offs between accuracy, timeliness, and complexity. Teams often fall into the trap of treating aggregated telemetry tables as authoritative sources for capacity or billing decisions, which can lead to significant issues.

Firstly, let's clarify what we mean by "observability warehouse." It is a data architecture that captures operational events from various sources like logs, metrics, traces, etc., and stores them in a form that allows for analysis. In contrast, a system-of-record typically refers to the authoritative source of truth for critical business operations.

When teams treat an observability warehouse as authoritative, several problems can arise:

Data Freshness: Observability warehouses are often designed for aggregation and long-term storage, not real-time decision-making. Aggregated data may be outdated by the time it is useful.
Data Integrity: Aggregation processes introduce potential inaccuracies or loss of details that might be critical for certain decisions.
Latency Issues: The delay between event capture and analysis in an observability warehouse can cause discrepancies in real-time decision-making.

Now, let's go through the reasoning step by step:

Step-by-Step Reasoning

Understanding the Nature of Observability Warehouses:

Purpose: Observability warehouses are designed to store vast amounts of operational data for long-term analysis and root cause determination.
Data Characteristics: They contain detailed, granular events that provide a complete picture of system behavior over time.

Role of Aggregated Data in Decision Making:

Timeliness: Real-time decisions require timely data. Aggregated telemetry tables are often several minutes or even hours old by the time they are useful for decision-making.
Detail Loss: Aggregation processes reduce the granularity of data, potentially leading to loss of critical details that might be important for certain decisions.

Impact on Critical Decisions:

Capacity Planning: Relying solely on aggregated telemetry tables can lead to under-provisioned or over-provisioned resources due to inaccurate capacity estimates.
Billing and Cost Management: Pricing based on historical averages may not accurately reflect current usage, leading to financial misalignment.

Best Practices:

Separation of Concerns: Keep the observability warehouse for long-term analysis while using real-time systems (like event streams) for critical decision-making.
Incremental Aggregation: Implement incremental aggregation processes that update the observability warehouse in near-real time, minimizing latency.

#### Closing Summary

In summary, an observability warehouse should not be treated as a system-of-record for any decisions requiring immediate and accurate data. It is best suited for long-term analysis and root cause determination. Using it as a source of truth for real-time decision-making can lead to significant issues such as outdated data, loss of detail, and inaccurate outcomes.

#### Common Mistakes
and Follow-Ups
Mistake 1: Treating aggregated telemetry tables as authoritative sources for real-time decisions.
Mistake 2: Overlooking the need for near-real-time systems that can handle high-frequency data.

#### Follow-Up
Questions:
How do you balance the trade-offs between detailed event storage and aggregated analysis?
What tools or techniques would you use to ensure both timely and accurate data in a real-time decision-making system?

**Comparison:**

| Timestamp | Metric Name | Value | Source ID |
|-----------|-------------|-------|-----------|
| 2023-10-01T10:00:00Z | cpu_usage | 75 | server1 |
| 2023-10-01T10:05:00Z | memory_usage | 80 | server1 |

**Code:**

```python
# Example of incremental aggregation process
def update_observability_warehouse(event):
    # Logic to update the observability warehouse with the latest event details
    pass

# Simulating an event stream
event_stream = [
    {"timestamp": "2023-10-01 10:00", "metric_value": 10},
    {"timestamp": "2023-10-01 10:05", "metric_value": 15}
]

for event in event_stream:
    update_observability_warehouse(event)
```

**Example input / output:**

```text
Input:
Event stream with CPU and memory usage metrics

Output:
Updated observability warehouse with detailed events, ready for long-term analysis
```

**Data:**

```json
{
  "events": [
    {
      "timestamp": "2023-10-01T10:00:00Z",
      "metric_name": "cpu_usage",
      "value": 75,
      "source_id": "server1"
    },
    {
      "timestamp": "2023-10-01T10:05:00Z",
      "metric_name": "memory_usage",
      "value": 80,
      "source_id": "server1"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Event Stream
4. Real-time Processing?
5. Incrementally UpdateWarehouse
6. Late Aggregation toWarehouse

