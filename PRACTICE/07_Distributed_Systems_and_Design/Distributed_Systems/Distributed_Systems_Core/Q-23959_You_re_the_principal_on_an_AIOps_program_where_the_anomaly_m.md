---
type: "practice"
difficulty: "Deep"
---

# You're the principal on an AIOps program where the anomaly model's own training data comes from a distributed feature pipeline that lagged, so it learned on stale telemetry and started false-alarming. Explain the freshness SLI you'd put on the feature path and why staleness here is a correctness bug, not a cost one.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Clock Skew and Time Windows](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In distributed systems, ensuring data freshness is crucial to avoid false positives in anomaly detection models. In our AIOps scenario, the anomaly model trained on stale telemetry led to incorrect alerts. To address this, we need a SLI (Service Level Indicator) that measures the freshness of feature data and ensure it aligns with the system's requirements.

#### Triaging the Question

The question is probing how well you can define and measure data freshness in a distributed system. Specifically, we're looking at:

What exactly is the SLI for data freshness?
Why is staleness here considered a correctness issue rather than a cost issue?

#### First-Principles Solution
Defining Data Freshness SLI

To tackle this problem, let's define the SLI as follows:

SLI (Data Freshness): The percentage of feature data that has been updated within the last (T) minutes.

Here, (T) can be set based on the expected latency of the distributed system and the desired freshness level. For instance, if we expect telemetry to be ingested every 5 minutes, a reasonable threshold might be (T = 30).

Why Staleness is a Correctness Issue

In anomaly detection models, data staleness can lead to incorrect predictions because:

Anomaly Detection Relying on Outdated Information: The model learns from stale telemetry and makes decisions based on outdated information.
False Positives/Negatives: Stale data may result in false alarms or missed anomalies, leading to suboptimal operational decision-making.
Measuring Data Freshness

To measure freshness, we can use the following steps:

Track Feature Updates:
Maintain a timestamp for each feature update.
Calculate Latency Distribution:
For each feature, calculate the difference between the current time and its last update time.
Apply SLI Formula:
Compute the percentage of features updated within (T) minutes.

#### Closing Summary

To ensure our anomaly model's data is fresh, we define a SLI based on the percentage of feature updates within a given timeframe. This approach directly addresses correctness issues by ensuring that the model operates on up-to-date information, thereby reducing false alarms and improving overall system reliability.

#### Common Mistakes/Follow-Ups
Ignoring Edge Cases: Overlooking edge cases where data staleness might still occur despite SLI thresholds.
Not Considering System Load Impact: Failing to account for the potential load impact of frequent updates on the distributed pipeline.
Inadequate Threshold Selection: Choosing a (T) value that is too short or long, leading to either overly aggressive freshness checks or insufficient data quality.

#### Example
Calculation

Let's consider an example:

Suppose we have 10 features, and 7 out of these 10 were updated within the last 30 minutes.
The SLI would be ( \frac{7}{10} * 100 = 70% ).

This indicates that 70% of our feature data is fresh.

**Comparison:**

```
Feature	Last Update Time	Freshness Status
CPUUsage	2023-09-15T14:30:00+00:00	Stale
MemoryUsage	2023-09-15T14:28:00+00:00	Fresh
NetworkIngress	2023-09-15T14:35:00+00:00	Fresh
DiskReadSpeed	2023-09-15T14:27:00+00:00	Fresh
Feature	Freshness Status
CPUUsage	Stale
MemoryUsage	Fresh
NetworkIngress	Fresh
DiskReadSpeed	Fresh
```

**Code:**

```python
def calculate_data_freshness(features, threshold_minutes=30):
    current_time = datetime.now()
    updated_features = [feature for feature in features if (current_time - feature['last_update']).total_seconds() <= threshold_minutes * 60]
    return (len(updated_features) / len(features)) * 100
```

**Example input / output:**

```text
Given the above data and threshold of 30 minutes:
- CPUUsage was updated at 14:30 (5 minutes ago)
- MemoryUsage was updated at 14:28 (2 minutes ago)
- NetworkIngress was updated at 14:35 (0 minutes ago)
- DiskReadSpeed was updated at 14:27 (3 minutes ago)

Calculate:
- Updated features: ["MemoryUsage", "NetworkIngress", "DiskReadSpeed"]
- Total updated percentage = (3/4) * 100 = 75%
```

**Data:**

```json
[
    {"name": "CPUUsage", "last_update": datetime(2023, 9, 15, 14, 30)},
    {"name": "MemoryUsage", "last_update": datetime(2023, 9, 15, 14, 28)},
    {"name": "NetworkIngress", "last_update": datetime(2023, 9, 15, 14, 35)},
    {"name": "DiskReadSpeed", "last_update": datetime(2023, 9, 15, 14, 27)}
]
```

**Flow:**

1. Yes
2. No
3. Feature Data
4. Is Last Update within T?
5. Fresh
6. Stale

