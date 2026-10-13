---
type: "practice"
difficulty: "Deep"
---

# You detect that anomaly recall silently dropped for one customer segment while aggregate detection metrics look healthy. Explain why averaged metrics hide per-segment tail behavior and how you'd instrument distributed monitoring to surface the per-segment degradation.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Aggregation Hiding Tail Problems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Aggregation_Hiding_Tail_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed system, it's common to rely on aggregated metrics like anomaly recall rates to get an overall view of system health. However, these metrics can mask significant issues within specific segments or subsets of users. In your scenario, while the aggregate detection metrics look healthy, there’s a silent drop in anomaly recall for one customer segment. This problem is due to how averaged metrics obscure tail behavior and the need for granular monitoring.

To address this issue, we'll instrument distributed monitoring to surface per-segment degradation by implementing custom monitoring practices that account for variance within segments. We’ll use first-principles reasoning to break down the solution step-by-step.

**Comparison:**

| Segment | Label   | Anomalies Detected | Count | Total Anomalies | Recall Rate (%) |
|---------|---------|--------------------|-------|-----------------|----------------|
| A       | label_1 | True               | 1     | 1               | 66.67          |
| A       | label_2 | False              | 0     | 0               | 0.0            |
| A       | label_3 | True               | 1     | 1               | 66.67          |

**Code:**

```python
# Example of a basic structure for segment-specific metrics collection
class SegmentMetricsCollector:
    def __init__(self, customer_segment):
        self.segment = customer_segment
        self.recall_counts = {}

    def record_anomaly(self, anomaly_detected, label):
        if label not in self.recall_counts:
            self.recall_counts[label] = {'count': 0, 'total_anomalies': 0}

        if anomaly_detected:
            self.recall_counts[label]['count'] += 1
            self.recall_counts[label]['total_anomalies'] += 1

    def get_recall_rate(self):
        total_count = sum(count['count'] for count in self.recall_counts.values())
        total_anomalies = sum(count['total_anomalies'] for count in self.recall_counts.values())

        if total_count == 0:
            return 0.0

        recall_rate = (total_anomalies / total_count) * 100
        return round(recall_rate, 2)
```

**Example input / output:**

```text
# Example input and output for the SegmentMetricsCollector class
collector = SegmentMetricsCollector("Segment A")
collector.record_anomaly(True, "label_1")
collector.record_anomaly(False, "label_2")
collector.record_anomaly(True, "label_3")

print(collector.get_recall_rate())  # Output: 66.67
```

**Data:**

```yaml
# Sample data schema for segment-specific metrics
- customer_segment: "Segment A"
  anomalies_detected: [True, False, True]
  labels: ["label_1", "label_2", "label_3"]
```

**Flow:**

1. Start
2. Initialize Metrics Collector
3. Record Anomaly Detected
4. Update Count and Total
5. Calculate Recall Rate
6. Generate Report
7. Display Per-Segment Metrics
8. Monitor for Degradation

