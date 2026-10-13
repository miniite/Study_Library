---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a data incident and a data outage, and how do you set severity levels for data quality issues?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Quality Monitoring](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this response, I'll explain the difference between a data incident and a data outage in the context of data observability. We'll also delve into setting severity levels for data quality issues. By understanding these concepts, you can effectively manage and mitigate risks associated with your data pipelines.

Data incidents and outages are two critical aspects of data observability that help us understand the state of our data systems. An incident refers to a short-lived event where something has gone wrong but is being actively resolved. On the other hand, an outage signifies a longer-term service disruption where the system is not functioning as expected. The severity levels for these issues are determined based on their impact and urgency.

Firstly, let's define what constitutes an incident:

An incident typically occurs when there's a deviation from the expected data quality or availability but can be quickly addressed.

#### Example
s include: temporary data processing delays, minor data schema changes that don't affect critical operations, or intermittent network issues causing data latency.

In contrast, outages are more severe and impactful:

An outage is characterized by prolonged periods where systems are unable to function as intended.

#### Example
s include: complete loss of data access for hours, critical data missing from the system, or total shutdown of a data pipeline due to a significant error.

When setting severity levels, we need to consider several factors such as business impact, user experience, and time-to-resolution. Here’s how we can categorize these issues:

Critical: High business impact, immediate action required (e.g., outages affecting multiple systems or services).
Severe: Significant but not immediate; requires urgent attention (e.g., data processing delays with no workaround in place).
Moderate: Substantial impact, but time allows for some mitigation efforts.
Low: Minimal impact, often managed through regular monitoring and maintenance.

To effectively manage these issues, we can implement a structured approach:

Detection: Use tools like anomaly detection systems to flag deviations from expected data patterns.
Response: Establish clear protocols for incident response, ensuring that alerts are sent to the right team members.
Resolution: Prioritize critical incidents and ensure they are resolved quickly to minimize business impact.
Post-Event Review: Conduct post-mortems to identify root causes and implement preventive measures.

#### Common mistakes
often arise from underestimating the severity of issues, leading to delayed responses or lack of proper documentation. It’s crucial to have a robust monitoring and alerting system in place that can help us quickly identify and address these issues before they escalate into more severe outages.

**Comparison:**

```
Incident Description	Severity Level	Impact
Complete loss of data access for users across multiple services.	Critical	High business impact, critical systems unavailable.
Data processing delays with no immediate workaround.	Severe	Significant operational disruption but manageable within a short timeframe.
Minor data quality issues affecting single service.	Moderate	Substantial impact, requires attention and mitigation efforts.
```

**Code:**

```python
# Example Python code for anomaly detection
def detect_anomaly(data):
    # Simple moving average calculation as an example
    avg = sum(data) / len(data)
    deviation = max(abs(x - avg) for x in data)
    if deviation > threshold:
        return True, "Anomaly detected"
    else:
        return False, None
```

**Example input / output:**

| Incident Description | Severity Level |
|----------------------|---------------|
| Critical outage: Complete loss of data access for users across multiple services. | Critical |
| Severe incident: Data processing delays with no immediate workaround. | Severe |
| Moderate incident: Minor data quality issues affecting single service. | Moderate |

**Data:**

```json
{
  "data_quality_issues": [
    {
      "type": "critical",
      "description": "Critical outage: Complete loss of data access for users across multiple services.",
      "impact": "High business impact, critical systems unavailable."
    },
    {
      "type": "severe",
      "description": "Severe incident: Data processing delays with no immediate workaround.",
      "impact": "Significant operational disruption but manageable within a short timeframe."
    },
    {
      "type": "moderate",
      "description": "Moderate incident: Minor data quality issues affecting single service.",
      "impact": "Substantial impact, requires attention and mitigation efforts."
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Detect Anomaly
4. Is Anomaly Detected?
5. Alert Team
6. Monitor Continuously
7. Respond to Incident
8. Resolve Issue
9. Post-Event Review

