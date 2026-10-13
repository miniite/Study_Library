---
type: "practice"
difficulty: "Deep"
---

# How do you balance real-time alerting against alert noise when a data source has legitimate spikes during sales events?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distribution Change Alerts](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of Data Engineering, real-time observability is crucial to ensure that issues in your data pipeline are caught and addressed quickly. However, balancing real-time alerting with avoiding noise from legitimate spikes (like during sales events) can be a delicate task. This question involves understanding how to set up effective alerting systems while minimizing false positives. Let's break it down step-by-step.

#### Triaging the Question

The core of this question is about managing alerts in a way that they only trigger on genuine issues, without overwhelming the team with noise from predictable events like sales spikes. The key is understanding the nature of the data and its patterns, as well as setting up robust alerting mechanisms.

#### Assumptions
Sales Events: These are periodic, known times when you expect a significant increase in data volume or activity.
Real-time Data Processing: You have a real-time pipeline that processes incoming data quickly and can generate alerts based on this data.
Historical Data: There is historical data available to understand the patterns of legitimate spikes.

#### First-Principles Solution

Analyze Historical Data:

Use past sales event data to determine typical volume increases.
Identify patterns in the data, such as specific times or types of events that cause spikes.

Set Thresholds Based on Patterns:

Define thresholds for alerts based on historical data analysis.
For example, if you see a 50% increase during sales events, set your threshold to trigger only when there's a 75% or greater increase, thus ignoring the normal spikes.

Implement Time-Based Filtering:

Use time windows to filter out alerts during known event times.
For instance, if you know a sale starts at 9 AM and ends at 12 PM on certain days, configure your alert system to ignore data during these periods.

Use Statistical Methods:

Apply statistical methods like Z-scores or anomaly detection to identify genuine outliers.
These techniques can help distinguish between noise and actual issues that need attention.

Review Alerts in Context:

Implement a review process for alerts, especially those generated during known events.
This can involve setting up secondary checks where an alert is only considered true if it meets multiple criteria (e.g., persistence over time).

Feedback Loop:

Continuously refine your thresholds and filtering rules based on feedback from the team.
Monitor which alerts are genuine issues and adjust accordingly.

#### Closing Summary

Balancing real-time alerting with legitimate data spikes requires a multifaceted approach. By analyzing historical data, setting intelligent thresholds, using time-based filters, applying statistical methods, and implementing a review process, you can minimize noise while ensuring that critical issues are caught promptly.

#### Common Mistakes/Follow-Ups
Over-Reliance on Thresholds: Relying solely on static thresholds without considering the context of data spikes.
Lack of Historical Data: Not having enough historical data to set meaningful thresholds and patterns.
Ignoring Team Feedback: Failing to incorporate feedback from operations teams, which can provide insights into what is truly an issue.
Triaging Follow-Ups
How would you handle false negatives in your system?
Can you walk me through a specific example of how you've implemented these strategies in the past?
What tools or technologies do you use for statistical analysis and anomaly detection?

**Comparison:**

```
Date	Increase %	Volume
2023-10-05	60	150,000
2023-11-08	70	180,000
```

**Code:**

```python
# Example Python code to illustrate setting thresholds based on historical data

def analyze_sales_events(historical_data):
    # Analyze sales events to determine typical volume increases
    avg_increase = calculate_average_increase(historical_data)
    return avg_increase

def set_alert_threshold(avg_increase):
    # Set threshold for alerts, considering normal spikes
    if avg_increase > 50:
        return avg_increase * 1.25
    else:
        return avg_increase * 1.5

def process_real_time_data(real_time_data, threshold):
    # Process real-time data and compare against threshold
    if real_time_data > threshold:
        send_alert(real_time_data)
```

**Example input / output:**

```text
historical_data = [
    {"date": "2023-10-05", "increase_percentage": 60, "volume": 150000},
    {"date": "2023-11-08", "increase_percentage": 70, "volume": 180000}
]

avg_increase = analyze_sales_events(historical_data)
threshold = set_alert_threshold(avg_increase)

print(f"Average Increase: {avg_increase}, Alert Threshold: {threshold}")
```

**Data:**

```json
{
  "historical_sales_events": [
    {
      "date": "2023-10-05",
      "increase_percentage": 60,
      "volume": 150000
    },
    {
      "date": "2023-11-08",
      "increase_percentage": 70,
      "volume": 180000
    }
  ]
}
```

**Flow:**

1. Analyze Historical Data
2. Set Threshold
3. Process Real-Time Data
4. Generate Alert if Necessary
5. Review and Adjust
6. Feedback Loop

