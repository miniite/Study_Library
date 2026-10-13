---
type: "practice"
difficulty: "Deep"
---

# How would you set up alerting on the gateway so a single upstream going from 0.1% to 5% error rate pages on-call fast, but a slow drift from 0.1% to 0.3% doesn't; what multi-window burn-rate setup do you use?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Connection Reuse and Client Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up alerting on an API Gateway so that it responds quickly to significant changes in error rates while ignoring minor drifts, we need a robust and nuanced approach that balances sensitivity with stability. This involves understanding the baseline error rate, setting thresholds for alerts based on both absolute values and burn-rate over time, and implementing multi-window monitoring.

#### Triaging the Question

The core of this question is to design an alerting mechanism that can detect sudden increases in error rates while avoiding false positives from small, gradual changes. The interviewer is probing our understanding of statistical anomaly detection, threshold setting, and temporal awareness in alert systems.

#### Assumptions
We are using a monitoring tool like Prometheus or New Relic.
We have access to detailed error rate metrics for the upstream services.
The API Gateway is integrated with a centralized alerting system.

#### Solution
Step 1: Baseline Error Rate Estimation

First, we need to establish a baseline error rate. For an API Gateway that sees minimal traffic (e.g., ~0.1% error rate), this baseline can be established over a period where the service is known to operate normally.

BASH
# Example command to calculate baseline
aws lambda invoke --function-name getBaselineErrorRate --log-type Tail --output text | grep 'error rate' | awk '{print $3}'
Step 2: Thresholds for Alerting

For significant changes, we can set thresholds based on both absolute error rates and burn-rate (change in error rate over time). For example:

High Alert: When the error rate exceeds 5%.
Critical Alert: If there's a sudden increase from 0.1% to 5% or more within a short period, trigger immediate alerts.

For minor drifts, we might use a burn-rate approach with multiple windows:

Slow Drift Window (24 hours): Monitor the error rate over 24-hour intervals.
High Alert Window (1 hour): Trigger an alert if there's a sudden increase in error rate within this window.
Step 3: Multi-Window Burn-Rate Setup

To implement multi-window burn-rate, we can use a statistical approach to detect anomalies. For instance, we can use the Z-score or moving averages to identify significant deviations from the baseline.

PYTHON
def calculate_z_score(current_rate, baseline_rate, std_dev):
    return (current_rate - baseline_rate) / std_dev

def monitor_error_rates(error_rates, baseline_rate, std_dev):
    z_scores = [calculate_z_score(rate, baseline_rate, std_dev) for rate in error_rates]
    if any(z > 3 for z in z_scores):  # Z-score threshold for high alert
        return 'High Alert'
    elif any(z > 1.5 for z in z_scores[-24:]):  # Z-score threshold for slow drift window
        return 'Slow Drift Alert'
    else:
        return 'No Alert'

# Example usage
error_rates = [0.1, 0.15, 0.2, 0.3, 0.5]
baseline_rate = 0.1
std_dev = 0.05

result = monitor_error_rates(error_rates, baseline_rate, std_dev)
print(result)
Step 4: Implementing Alerts in Prometheus

Using Prometheus as the monitoring tool, we can set up rules that trigger alerts based on the calculated metrics.

PROMETHEUS
# High Alert Rule
ALERT SuddenErrorRateIncrease
  IF error_rate > 5%
  FOR 1m
  LABELS {severity="critical"}
  ANNOTATIONS {summary="Error rate suddenly increased"}

# Slow Drift Rule
ALERT SlowDriftErrorRate
  IF (error_rate - mean(error_rate[24h])) / std_dev(error_rate) > 1.5
  FOR 1m
  LABELS {severity="warning"}
  ANNOTATIONS {summary="Error rate slowly drifting up"}

#### Closing Summary
and Common Mistakes

By using a combination of absolute error rates, burn-rate analysis over multiple windows, and threshold-based alerts in Prometheus, we can achieve robust alerting that quickly identifies significant issues while ignoring minor fluctuations. Common mistakes include:

Overly sensitive thresholds leading to false positives.
Lack of context-aware alerts (e.g., ignoring baseline adjustments).
Not considering the time window for burn-rate calculations.

#### Follow-Ups
How would you handle noisy error rates?
Can you integrate this with other monitoring tools like Grafana or Splunk?
What measures do you take to ensure the alerting system itself is reliable?

**Comparison:**

```
Window	Error Rate (Z-score)
1 hr	-4
24 hrs	-3
48 hrs	-2
```

**Code:**

```python
def calculate_z_score(current_rate, baseline_rate, std_dev):
    return (current_rate - baseline_rate) / std_dev

def monitor_error_rates(error_rates, baseline_rate, std_dev):
    z_scores = [calculate_z_score(rate, baseline_rate, std_dev) for rate in error_rates]
    if any(z > 3 for z in z_scores):  # Z-score threshold for high alert
        return 'High Alert'
    elif any(z > 1.5 for z in z_scores[-24:]):  # Z-score threshold for slow drift window
        return 'Slow Drift Alert'
    else:
        return 'No Alert'
```

**Example input / output:**

```text
Input:
Error rates: [0.1, 0.15, 0.2, 0.3, 0.5]
Baseline rate: 0.1
Standard deviation: 0.05

Output:
'High Alert'
```

**Data:**

```json
{
  "error_rates": [0.1, 0.15, 0.2, 0.3, 0.5],
  "baseline_rate": 0.1,
  "std_dev": 0.05
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Error Rate Measurement
6. Baseline Established?
7. Moving Average Calculation
8. Z-Score Calculation
9. Threshold Check (High Alert)
10. Slow Drift Window Check
11. Trigger Slow Drift Alert
12. No Alert

