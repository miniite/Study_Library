---
type: "practice"
difficulty: "Deep"
---

# What's your approach to governing sampling policies for traces so cost is bounded but the rare-but-critical error traces AIOps needs are never sampled away?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Platform Telemetry](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Platform_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern sampling policies for traces in an AIOps environment, ensuring that cost is bounded while retaining critical error traces, we need to strike a balance between efficiency and accuracy. This involves understanding the nature of trace data, defining acceptable error rates, and implementing sampling strategies that are both cost-effective and reliable.

#### Triaging the Question

The core of this question revolves around managing trace sampling policies in an AIOps system. The primary goals are:

Cost Management: Minimize the storage and processing costs associated with trace data.
Critical Error Retention: Ensure that important error traces, which could indicate critical issues, are not discarded during sampling.

#### Assumptions
We have a large-scale distributed application generating numerous trace logs.
The system has limited storage capacity and computational resources.
There is a known distribution of events: most are normal operations, while some are rare but critical errors that need to be retained.

#### First Principles Solution
1. Define the Criticality of Events

First, we need to categorize events based on their importance:

Normal Operations: These can be sampled heavily without affecting system performance.
Critical Errors: These should always be retained and not discarded under any circumstances.
2. Implement Hierarchical Sampling

We will implement a hierarchical sampling strategy that considers the criticality of each trace:

High-Risk Traces (Critical Errors): Sample these traces fully to ensure no data loss.
Medium-Risk Traces: Apply a lower sampling rate but still retain enough data for analysis.
Low-Risk Traces (Normal Operations): Use aggressive sampling rates to minimize storage costs.
3. Define Sampling Ratios

Based on the distribution of events, we can define appropriate sampling ratios:

For high-risk traces, use 100% sampling.
For medium-risk traces, use a 5-20% sampling rate.
For low-risk traces, consider using 90-99.9% sampling rates.
4. Real-Time Monitoring and Feedback

Implement real-time monitoring to track the effectiveness of the sampling strategy:

Feedback Mechanism: Periodically review the sampling results to ensure critical errors are not being missed.
Adjustment Strategy: Automatically adjust sampling ratios based on detected anomalies or feedback from users.

#### Implementation
Example

Suppose we have an application that generates 10 million trace events per day. We can categorize them as follows:

95% low-risk normal operations
4% medium-risk warning signs
1% high-risk critical errors

Using a sampling strategy, we might set the following parameters:

PYTHON
# Sampling configuration
sampling_config = {
    "critical_errors": {"sampling_ratio": 100, "storage_limit": None},
    "warnings": {"sampling_ratio": 20, "storage_limit": 500000},
    "normal_operations": {"sampling_ratio": 98.95, "storage_limit": 7000000}
}
Dry Run

Let's run a sample input to see how the sampling would work:

PYTHON
input_events = [1, 2, 3, ..., 10000000] # 10M events

# Apply sampling based on configuration
def apply_sampling(events):
    critical_errors = [event for event in events if is_critical(event)]
    warnings = [event for event in events if is_warning(event)]
    normal_operations = [event for event in events if not is_critical(event) and not is_warning(event)]

    sampled_events = []

    # Sample critical errors fully
    sampled_events.extend(critical_errors)

    # Sample warnings with 20% rate
    sample_size = int(len(warnings) * 0.2)
    sampled_events.extend(random.sample(warnings, sample_size))

    # Sample normal operations with 98.95% rate
    sample_size = int(len(normal_operations) * 0.9895)
    sampled_events.extend(random.sample(normal_operations, sample_size))

    return sampled_events

sampled_output = apply_sampling(input_events)
print(f"Sampled Events: {len(sampled_output)}")

#### Closing Summary
and Common Mistakes/Follow-Ups

To govern sampling policies effectively in an AIOps environment:

Hierarchical Sampling: Ensure critical errors are always retained.
Real-Time Monitoring: Continuously monitor the effectiveness of your sampling strategy.
Feedback Mechanism: Adjust sampling ratios based on detected anomalies.

#### Common mistakes
include over-sampling normal operations, under-sampling warnings, and failing to account for varying event distributions. Always validate that rare but critical errors are not being missed.

#### Follow-ups
could involve discussing advanced techniques like adaptive sampling or integrating machine learning models to dynamically adjust sampling rates based on anomaly detection.

**Comparison:**

```
Category	Sampling Ratio	Storage Limit
Critical Errors	100%	None
Warnings	20%	500,000
Normal Operations	98.95%	7,000,000
```

**Code:**

```python
# Sampling configuration example
sampling_config = {
    "critical_errors": {"sampling_ratio": 100, "storage_limit": None},
    "warnings": {"sampling_ratio": 20, "storage_limit": 500000},
    "normal_operations": {"sampling_ratio": 98.95, "storage_limit": 7000000}
}

# Sample input events
def apply_sampling(events):
    critical_errors = [event for event in events if is_critical(event)]
    warnings = [event for event in events if is_warning(event)]
    normal_operations = [event for event in events if not is_critical(event) and not is_warning(event)]

    sampled_events = []

    # Sample critical errors fully
    sampled_events.extend(critical_errors)

    # Sample warnings with 20% rate
    sample_size = int(len(warnings) * 0.2)
    sampled_events.extend(random.sample(warnings, sample_size))

    # Sample normal operations with 98.95% rate
    sample_size = int(len(normal_operations) * 0.9895)
    sampled_events.extend(random.sample(normal_operations, sample_size))

    return sampled_events

# Example input data
input_events = [1, 2, 3, ..., 10000000] # 10M events

sampled_output = apply_sampling(input_events)
print(f"Sampled Events: {len(sampled_output)}")
```

**Example input / output:**

```text
# Example input data
input_events = [1, 2, 3, ..., 10000000] # 10M events

# Apply sampling based on configuration
sampled_output = apply_sampling(input_events)

print(f"Sampled Events: {len(sampled_output)}")
```

**Data:**

```json
{
    "events": [
        {"id": 1, "category": "normal_operations"},
        {"id": 2, "category": "warnings"},
        {"id": 3, "category": "critical_errors"}
    ],
    "sampling_config": {
        "critical_errors": {"sampling_ratio": 100, "storage_limit": null},
        "warnings": {"sampling_ratio": 20, "storage_limit": 500000},
        "normal_operations": {"sampling_ratio": 98.95, "storage_limit": 7000000}
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is the Event Critical?
7. Sample Fully
8. Is the Event a Warning?
9. Sample 20% of Warnings
10. Sample 98.95% of NormalOperations
11. End
12. End
13. End

