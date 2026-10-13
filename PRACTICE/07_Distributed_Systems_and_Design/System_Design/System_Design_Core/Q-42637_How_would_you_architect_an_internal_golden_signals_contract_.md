---
type: "practice"
difficulty: "Deep"
---

# How would you architect an internal 'golden signals' contract so every service, regardless of team, emits comparable health telemetry? What do you standardize (semantic conventions) and how do you enforce it without policing?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a system that ensures every service, irrespective of the development team, emits comparable health telemetry, we need to establish a standardized contract or protocol known as "golden signals." These golden signals typically include latency, traffic rate (errors), and saturation metrics. By ensuring all services follow these conventions, we can create a unified view of system health across different services. This approach involves standardizing semantic conventions for metric names, units, labels, and how data is structured, while also finding ways to enforce adherence without creating unnecessary overhead.

The key idea here is to define a set of golden signals that are commonly accepted in the industry (e.g., latency, error rate, and saturation) and then specify how these metrics should be collected and reported. We need to ensure that each service adheres to these conventions to enable consistent monitoring and alerting across the system.

**Comparison:**

```
Metric Name	Unit	Description
latency_ms	Milliseconds	Average response time of the service
error_rate	Percent	Percentage of requests that result in errors
saturation	Percent	Percentage of CPU/memory usage
```

**Code:**

```python
# Example of a simplified golden signals contract in Python

class GoldenSignalsContract:
    def __init__(self, latency_ms, error_rate, saturation):
        self.latency_ms = latency_ms
        self.error_rate = error_rate
        self.saturation = saturation

    @staticmethod
    def validate(metric: dict) -> bool:
        required_keys = {"latency_ms", "error_rate", "saturation"}
        return set(metric.keys()) == required_keys and all(isinstance(v, (int, float)) for v in metric.values())

# Example usage
metric1 = {"latency_ms": 50, "error_rate": 0.02, "saturation": 80}
is_valid_metric1 = GoldenSignalsContract.validate(metric1)  # True

metric2 = {"latency_ms": 30, "error_rate": 0.04, "saturation": "high"}
is_valid_metric2 = GoldenSignalsContract.validate(metric2)  # False
```

**Example input / output:**

```text
Valid Metric1:
- Latency: 50ms
- Error Rate: 2%
- Saturation: 80%

Invalid Metric2:
- Latency: 30ms (should be an integer or float)
- Error Rate: 4%
- Saturation: "high" (should be a numeric value)
```

**Data:**

```json
{
    "metric1": {
        "latency_ms": 50,
        "error_rate": 0.02,
        "saturation": 80
    },
    "metric2": {
        "latency_ms": 30,
        "error_rate": 0.04,
        "saturation": "high"
    }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is metric validated?
5. Proceed to monitoring
6. Log error and retry validation
7. Metric added to telemetrystream
8. Alert developer and wait forcorrection

