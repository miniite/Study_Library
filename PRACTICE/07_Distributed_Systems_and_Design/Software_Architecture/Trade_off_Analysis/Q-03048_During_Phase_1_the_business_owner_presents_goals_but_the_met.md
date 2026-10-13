---
type: "practice"
difficulty: "Deep"
---

# During Phase 1 the business owner presents goals but the metrics they cite come from a dashboard the architect says is misleading. How do you resolve a disputed baseline before you let scenario elicitation proceed?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scenario Prioritization](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Scenario_Prioritization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a situation where the business owner and the architect have differing views on key metrics, it's crucial to establish a common ground for discussion. This involves resolving any potential disputes around the baseline before proceeding further in scenario elicitation. Here’s how you can approach this:

TL;DR: The goal is to validate the reliability of the provided metrics through critical analysis and possibly developing a more accurate baseline that both parties can agree upon.

#### Triage
The core issue here is ensuring the accuracy and relevance of the business owner's metrics. We need to determine if these metrics are based on sound data, whether they reflect the true state of the system, and how they impact key performance indicators (KPIs).

#### First-Principles Solution

Validate Data Sources: Begin by reviewing the origin and reliability of the dashboard's data sources. Are there any known issues or biases in the data collection methods?
Compare Metrics with Industry Standards: Evaluate if the metrics are comparable to industry standards or benchmarks.
Use Controlled Experiments: Implement controlled experiments to compare the dashboard’s metrics against real-world outcomes. This could involve setting up a small, controlled environment where both sets of metrics can be directly compared over time.
Develop a Shared Understanding: Engage in discussions with the business owner to understand their goals and assumptions behind the metrics. Use this understanding to collaboratively develop or validate new baseline metrics that align better with the system’s true performance.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Ensure all stakeholders are aligned on the importance of establishing a reliable baseline before moving forward.
Document any discrepancies found during validation and propose adjustments based on evidence gathered from experiments or data reviews.
Engage in iterative feedback loops to refine both metrics and assumptions as more data becomes available.

**Comparison:**

```
Time	Metric1 (A)	Metric2 (B)
2023-10-01	85	95
2023-10-02	90	100
```

**Code:**

```text
# Example code for controlled experiment setup
import pandas as pd

def compare_metrics(data1, data2):
    # Load and clean data
    df1 = pd.DataFrame(data1)
    df2 = pd.DataFrame(data2)

    # Normalize metrics if necessary
    normalized_df1 = normalize_data(df1)
    normalized_df2 = normalize_data(df2)

    # Compare metrics over time
    result = compare_over_time(normalized_df1, normalized_df2)

    return result

def normalize_data(df):
    # Implement normalization logic here
    pass

def compare_over_time(df1, df2):
    # Implement comparison logic here
    pass
```

**Example input / output:**

```text
data1 = [{"time": "2023-10-01", "metric_value": 85}, {"time": "2023-10-02", "metric_value": 90}]
data2 = [{"time": "2023-10-01", "metric_value": 95}, {"time": "2023-10-02", "metric_value": 100}]

result = compare_metrics(data1, data2)
print(result) # Output: Comparison results
```

**Data:**

```json
{
  "metrics1": [
    {"time": "2023-10-01", "metric_value": 85},
    {"time": "2023-10-02", "metric_value": 90}
  ],
  "metrics2": [
    {"time": "2023-10-01", "metric_value": 95},
    {"time": "2023-10-02", "metric_value": 100}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start Validation
4. Review Data Sources
5. Are Metrics Reliable?
6. Validate Against Benchmarks
7. Investigate Issues
8. Develop New Baseline
9. Adjust Metrics Based onFindings
10. Compare with Real-WorldOutcomes
11. Refine Assumptions
12. End

