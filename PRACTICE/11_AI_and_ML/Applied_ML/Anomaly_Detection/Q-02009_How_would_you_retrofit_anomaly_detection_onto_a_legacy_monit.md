---
type: "practice"
difficulty: "Intermediate"
---

# How would you retrofit anomaly detection onto a legacy monitoring stack that only supports static thresholds?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dynamic vs Static Alerting](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Dynamic_vs_Static_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrofit anomaly detection into a legacy monitoring stack that only supports static thresholds, we need to convert the static threshold approach into something more dynamic and adaptive. The key idea is to use statistical methods to dynamically set thresholds based on historical data patterns rather than fixed values. This involves identifying normal behavior and using it as a baseline for detecting anomalies.

First, let's triage the question:

What the interviewer is probing: Understanding how to integrate modern anomaly detection techniques into an older system without modifying its core functionalities.

#### Assumptions
The legacy monitoring stack works well but lacks adaptability. We will leverage statistical methods such as Z-score or moving average to dynamically adjust thresholds.

Now, let's reason through this step-by-step:

Data Collection and Baseline Estimation:

Collect historical data on the metrics being monitored.
Compute a baseline behavior using techniques like rolling mean or median over a recent time window.

Statistical Thresholds:

Use Z-score to identify anomalies: A metric is flagged as an anomaly if its Z-score exceeds a certain threshold (e.g., ±3).
Alternatively, use moving average and standard deviation to set thresholds dynamically.

#### Implementation
Strategy:

Implement a service or script that runs periodically.
Calculate the current statistical values using recent data.
Compare these values against pre-defined conditions to determine if an anomaly is present.
If an anomaly is detected, notify the system via its existing alert mechanisms (e.g., email, SMS).

Integration:

Integrate this service or script into the legacy monitoring stack without changing its core components.
Ensure it runs at a frequency that balances performance and accuracy.

Monitoring and Tuning:

Continuously monitor the system to ensure accurate anomaly detection.
Fine-tune parameters like window size, threshold values based on real-world outcomes.

#### Closing Summary
By leveraging statistical methods such as Z-scores or moving averages, we can dynamically adjust thresholds in a legacy monitoring stack that only supports static thresholds. This approach requires careful data handling and parameter tuning but provides significant improvements over static thresholding.

#### Common Mistakes

Failing to account for seasonal or periodic patterns in the data.
Setting overly strict thresholds leading to false positives.
Not continuously validating and adjusting parameters based on real-world performance.

