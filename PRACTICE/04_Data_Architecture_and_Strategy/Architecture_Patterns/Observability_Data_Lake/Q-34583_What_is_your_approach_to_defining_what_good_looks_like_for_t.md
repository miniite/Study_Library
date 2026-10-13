---
type: "practice"
difficulty: "Deep"
---

# What is your approach to defining what good looks like for the lake's query performance and how do you keep SLOs honest as data volume 10x's over three years?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ In-lake vs External Inference](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/In-lake_vs_External_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define what "good" looks like for the observability data lake's query performance, we need to establish Service Level Objectives (SLOs) that reflect both user expectations and operational feasibility. Over a period of three years, as the volume of data in the lake increases by 10x, maintaining these SLOs becomes critical. Here’s my approach:

Firstly, I would start with understanding the current state of the system—this includes analyzing historical query performance metrics, identifying bottlenecks, and assessing how well existing infrastructure scales under varying loads.

Secondly, based on this analysis, I will define a set of SLOs that cover key aspects such as response time, availability, and data freshness. These SLOs should be quantifiable and granular enough to provide actionable insights into the health of the system.

Thirdly, we need to implement monitoring and alerting mechanisms to continuously track these SLOs. This involves setting up automated checks for query performance metrics and triggering alerts when thresholds are breached. Additionally, we will use AIOps tools to correlate anomalies with other operational data, such as infrastructure metrics or user activity patterns.

Lastly, I would establish a feedback loop where the insights from monitoring help refine our SLOs over time. This iterative process ensures that our goals remain relevant and achievable as the system evolves.

**Comparison:**

```
Metric	Thresholds	Current Status
RT	≤ 200ms	Meets
A	99.5% availability	Meets
```

**Code:**

```python
# Example of defining query performance SLOs in code
class QueryPerformanceSLO:
    def __init__(self, max_response_time_ms, availability_percent):
        self.max_response_time_ms = max_response_time_ms
        self.availability_percent = availability_percent

    def check_slo(self, metric_data):
        response_times = [r["response_time"] for r in metric_data]
        availability = len([t for t in metric_data if "error" not in t]) / len(metric_data) * 100
        return all(r <= self.max_response_time_ms for r in response_times) and abs(availability - self.availability_percent) < 5
```

**Example input / output:**

```

Input:

Metric data for the past week

Output:

SLO check result (True if all metrics meet the SLOs)
Alert status (if any thresholds are breached)

Request:

JSON
Copy
{
    "metrics": [
        {"response_time": 150, "error": false},
        {"response_time": 220, "error": false},
        // more data points...
    ]
}

Response:

JSON
Copy
{
    "slo_check_result": true,
    "alert_status": false
}
```

**Data:**

```json
{
    "max_response_time_ms": 200,
    "availability_percent": 99.5
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Data Volume Increased?
5. Maintain SLOs with Monitoring
6. Iterate and Refine SLOs Basedon Feedback
7. End
8. Maintain Current SLOs

