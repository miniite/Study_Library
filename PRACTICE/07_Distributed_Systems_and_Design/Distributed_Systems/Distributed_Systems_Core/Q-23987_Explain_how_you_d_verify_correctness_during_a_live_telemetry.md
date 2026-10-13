---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd verify correctness during a live telemetry-backend migration you can't fully diff. Design a continuous shadow-query comparison and sampling pipeline, and describe how you'd triage the stream of legitimate-vs-bug metric discrepancies without stalling the cutover.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Verifying Live Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating a telemetry-backend system live, it's crucial to ensure correctness throughout the process, especially when full diffing isn't possible. In this scenario, we'll design a continuous shadow-query comparison and sampling pipeline to verify metrics integrity. This approach ensures that any discrepancies are identified and addressed without disrupting the migration.

First, let’s break down what we need:

Shadow Query Pipeline: A mechanism to run the new backend system in parallel with the old one.
Sampling Strategy: To handle large volumes of telemetry data efficiently.
Discrepancy Triage Mechanism: An automated and manual process to filter out legitimate vs. bug discrepancies.

Here's how we can achieve this from first principles:

TL;DR

We’ll set up a dual pipeline where the new backend runs alongside the old one, continuously sampling telemetry data for comparison. We'll implement an intelligent triage system that filters noise while alerting on significant discrepancies, ensuring minimal manual intervention during the migration.

#### Triaging the Question

The core challenge is to maintain correctness in a live environment without full diffing capabilities. The primary goals are:

Ensure Data Consistency: Verify that new and old backends produce equivalent results.
Efficient Monitoring: Handle large volumes of data without overwhelming resources or introducing significant latency.

#### Triage
Discrepancies: Identify real issues quickly while filtering out expected variations.

#### First Principles Solution
Shadow Query Pipeline

Parallel Execution:

The new backend runs alongside the old one for a period to gather telemetry data.
This ensures that the new system can be validated without disrupting service continuity.

Sampling Strategy:

Implement a stratified sampling approach where critical metrics (e.g., error rates, latency) are sampled more frequently than less impactful ones.
Use probabilistic algorithms like Reservoir Sampling to ensure unbiased data representation.
Continuous Comparison

Comparison Logic:

Define a set of baseline metrics and compare them continuously using statistical methods such as Z-scores or Chi-Squared tests.
Implement a tolerance threshold for acceptable discrepancies based on historical baselines.

Discrepancy Handling:

Log significant discrepancies with metadata (time, metric details) to facilitate quick analysis.
Use machine learning models to predict and filter out noise, reducing false positives.

#### Triage
Mechanism

Automated Alerts:

Set up automated alerts for metrics exceeding predefined thresholds.
These alerts should include severity levels based on the magnitude of discrepancy.

Manual Review:

Create a dashboard for DevOps teams to review and validate alerts.
Implement a triage workflow where urgent issues are escalated immediately while less critical ones are reviewed periodically.

#### Closing Summary

By setting up a robust shadow-query comparison pipeline with intelligent sampling, we can ensure that the new backend remains consistent with the old one. This approach minimizes disruption during the migration and ensures data integrity through automated and manual validation mechanisms.

#### Common Mistakes
and Follow-Ups
Overly Aggressive Sampling: Over-sampling can lead to resource exhaustion without providing additional value.
Insufficient Tolerance Thresholds: Setting overly strict thresholds may miss real issues, leading to false negatives.
Manual Interventions: Relying too heavily on manual reviews can introduce delays and human errors.

To address these concerns, it’s crucial to balance the sampling strategy with the tolerance threshold and ensure robust alert mechanisms are in place. Regularly reviewing and adjusting these parameters will help maintain a smooth migration process.

**Comparison:**

```
Metric	Old Value (Old Backend)	New Value (New Backend)	Discrepancy	Tolerance Threshold
Error Rate	0.012	0.013	0.001	±0.005
Latency	98ms	102ms	4ms	±20ms
```

**Code:**

```python
# Example of a simple sampling function using Reservoir Sampling
import random

def reservoir_sampling(data, sample_size):
    sample = []
    for i, item in enumerate(data):
        if len(sample) < sample_size:
            sample.append(item)
        else:
            k = random.randint(0, i)
            if k < sample_size:
                sample[k] = item
    return sample

# Example usage
sampled_data = reservoir_sampling(range(100), 20)
```

**Example input / output:**

```text
Input (old backend metric): {"metric": "error_rate", "value": 0.012}
Shadow query output (new backend metric): {"metric": "error_rate", "value": 0.013}

Discrepancy comparison result:
- Metric: error_rate
- Old Value: 0.012
- New Value: 0.013
- Discrepancy: |0.013 - 0.012| = 0.001

Threshold check:
- Tolerance Threshold for error_rate: ±0.005
- Is the discrepancy significant? No, as it's within the threshold.
```

**Data:**

```yaml
sampling_strategy:
  critical_metrics: [error_rate, latency]
  sampling_frequency:
    error_rate: every 5 minutes
    latency: every hour
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Metric Critical?
5. Sampling with ReservoirSampling
6. Comparison Logic
7. Discrepancy Triage
8. Triage Workflow
9. Triaged Metrics
10. Use Baseline Sampling

