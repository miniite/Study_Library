---
type: "practice"
difficulty: "Deep"
---

# Describe how you would build an SLO-based alerting strategy with burn-rate alerts instead of static latency thresholds.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Composite Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Composite_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: To build an SLO (Service Level Objective)-based alerting strategy using burn-rate alerts, we need to focus on monitoring the degradation in service quality over time rather than relying on static threshold values. This approach involves calculating the burn rate, which is the rate at which the service level is degrading, and setting up alerts based on this metric.

#### Triaging the Question
The interviewer is probing your understanding of SLOs, burn rates, and alerting strategies in AIOps tools like Datadog. They are also likely interested in how you would integrate these concepts to provide real-time insights into service health and proactively detect issues before they become critical.

#### Assumptions

The system has an established SLO for a specific SLI (Service Level Indicator).
You have access to historical data on the SLI.
The alerting strategy is designed to be dynamic, adapting to changes in service performance over time.

Step-by-Step Solution from First Principles:

Define the Service Level Objective: Start by defining what SLO means for your specific use case. For instance, if you are tracking response times, an SLO might be that 95% of requests should respond within 200ms.

Calculate Burn Rate: The burn rate is defined as the percentage of the SLO that has been breached over a given time period. If your SLO is 95th percentile ≤ 200ms and at some point, it starts to degrade such that 85% breaches the SLI threshold (i.e., more than 10% of requests take longer than 200ms), then you can calculate the burn rate as 10%.

Historical Data Analysis: Use historical data to establish a baseline and trend analysis. For example, if your system normally degrades at a rate of 5% per week, but suddenly starts degrading at 15%, this could indicate an underlying issue.

Set Dynamic Thresholds: Instead of using static thresholds (e.g., always alert when the 95th percentile is above 200ms), use dynamic thresholds that account for historical behavior. For instance, if your system typically degrades by 3% per week, setting an alert when the burn rate exceeds 10% could be more effective.

Implement Alerts: Use Datadog's capabilities to implement these alerts. Set up monitoring rules that trigger alerts based on calculated burn rates. Datadog allows for complex queries and dynamic thresholds, making it suitable for this approach.

Monitor and Adjust: Continuously monitor the performance and adjust the SLOs and burn rate calculations as necessary. This might involve redefining SLOs or adjusting how you calculate the burn rate based on feedback from system performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups: This approach to alerting using burn rates is more robust because it adapts to changes in service behavior, reducing false positives and missed alerts. However, a common mistake is over-reliance on historical data without considering recent trends or anomalies. Additionally, dynamic thresholds need to be carefully tuned to avoid noise.

#### Follow-ups
might include how you would handle noisy data, the impact of different SLO definitions, or integration with other monitoring tools.

**Comparison:**

| Time | Response Time |
|------|--------------|
| t1   | 250          |
| t2   | 300          |
| t3   | 200          |
| t4   | 190          |
| t5   | 210          |

**Code:**

```python
# Example Python code snippet for calculating burn rate and setting alerts

import pandas as pd

def calculate_burn_rate(df, sli_threshold):
    # Calculate 95th percentile response time
    p95 = df['response_time'].quantile(0.95)

    # Calculate breach percentage
    breaches = (df['response_time'] > sli_threshold).mean() * 100

    return breaches - sli_threshold

# Example usage with Datadog API integration
from datadog import initialize, api

options = {
    'api_key': '<YOUR_API_KEY>',
    'app_key': '<YOUR_APP_KEY>'
}

initialize(**options)

def set_alert(burn_rate):
    if burn_rate > 10:  # Define threshold for alerting
        print("Alert: Burn rate exceeds 10%")
        api.Event.create(title="SLO Breach", text=f"Burn rate is {burn_rate}%")

# Example DataFrame with response times
data = {
    'response_time': [250, 300, 200, 190, 210]
}
df = pd.DataFrame(data)

burn_rate = calculate_burn_rate(df, 200)
set_alert(burn_rate)
```

**Example input / output:**

```text
Input DataFrame:
+------------+
| response_time |
+------------+
|       250     |
|       300     |
|       200     |
|       190     |
|       210     |
+------------+

Output Burn Rate Calculation:
Burn rate = (80% breaches - 200ms threshold) - 200ms threshold
           = 60%
```

**Data:**

```json
{
    "data": {
        "response_time": [250, 300, 200, 190, 210]
    }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is SLO Breached?
5. Calculate Burn Rate
6. Set Alert if Burn Rate ExceedsThreshold
7. Continue Monitoring
8. Notify Operations Team

