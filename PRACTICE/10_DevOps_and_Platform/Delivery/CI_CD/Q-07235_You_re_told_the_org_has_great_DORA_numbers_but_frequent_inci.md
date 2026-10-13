---
type: "practice"
difficulty: "Expert"
---

# You're told the org has great DORA numbers but frequent incidents and low morale; what's your hypothesis for the disconnect and how do you look past vanity metrics to find the real delivery problem?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ DORA Metrics](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/DORA_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we see a discrepancy between high Digital Oxygenation Rate of Adoption (DORA) numbers—indicating frequent deployments and fast feedback loops—and frequent incidents and low morale among developers. This suggests that while automation and deployment processes are robust, there may be underlying issues in the development lifecycle that aren't being addressed by these metrics alone.

To probe beyond vanity metrics, we need to look at the full CI/CD pipeline and its integration with other aspects of the software development process. Let's break down the problem from first principles:

Deployment Frequency vs. Stability: High deployment frequency might indicate a culture that prioritizes speed over quality. Frequent deployments without proper testing can lead to more frequent incidents.
Feedback Loops and Defect Detection: Fast feedback mechanisms are crucial, but they must be effective. If the feedback loops are too short or insufficiently detailed, developers may not have enough time or information to address issues properly.
Developer Morale and Work Environment: Low morale could stem from various factors such as excessive work pressure, lack of autonomy, or feeling that their efforts aren't valued.

To identify the root causes:

Review Incident Data: Analyze incident logs to determine if there are common patterns (e.g., related to specific features or types of changes). This can help pinpoint areas where additional testing or documentation might be needed.
Conduct Developer Surveys and Interviews: Gather qualitative feedback from developers about their experience with the CI/CD pipeline, workload, and overall satisfaction. This will provide insights into any pain points they face.
Examine Deployment Logs: Look at detailed logs of deployments to check for failures or anomalies that might not be captured in DORA metrics.
Evaluate Test Coverage and Automation: Ensure that test coverage is comprehensive enough to catch issues before they make it to production.

By combining quantitative data with qualitative insights, we can build a more holistic picture of the development process and identify areas for improvement.

**Comparison:**

```
Metrics	Value
Deployment Frequency	High (DORA)
Incident Frequency	High
Morale	Low
Developer Feedback	Percentage
Autonomy	30%
Workload	45%
Test Coverage	20%
Documentation	5%
```

**Code:**

```text
# Example code snippet for analyzing deployment logs
def analyze_deployment_logs(logs):
    failed_deployments = 0
    total_deployments = len(logs)

    for log in logs:
        if "failure" in log:
            failed_deployments += 1

    failure_rate = (failed_deployments / total_deployments) * 100
    return failure_rate

# Sample data
logs = [
    {"timestamp": "2023-10-01", "result": "success"},
    {"timestamp": "2023-10-02", "result": "failure"},
    # ... more logs ...
]

print(analyze_deployment_logs(logs))  # Output: failure_rate
```

**Example input / output:**

```text
# Example input and output for analyzing deployment logs
logs = [
    {"timestamp": "2023-10-01", "result": "success"},
    {"timestamp": "2023-10-02", "result": "failure"},
    {"timestamp": "2023-10-03", "result": "success"}
]

failure_rate = analyze_deployment_logs(logs)
print(f"Failure Rate: {failure_rate}%")  # Output: Failure Rate: 33.33%
```

**Data:**

```json
{
  "logs": [
    {"timestamp": "2023-10-01", "result": "success"},
    {"timestamp": "2023-10-02", "result": "failure"},
    {"timestamp": "2023-10-03", "result": "success"}
  ]
}
```

**Flow:**

1. DORA numbers
2. Incident Logs
3. Developer Surveys
4. Start
5. Identify Key Metrics
6. Determine DeploymentFrequency
7. Frequent Incidents
8. Low Morale
9. Analyze Deployment Logs forFailures
10. Conduct Root Cause Analysisof Incidents
11. Review Feedback fromDevelopers
12. Implement Improvements inCI/CD Pipeline
13. Enhance Test Coverage andAutomation
14. Adjust Workload andAutonomy Policies

