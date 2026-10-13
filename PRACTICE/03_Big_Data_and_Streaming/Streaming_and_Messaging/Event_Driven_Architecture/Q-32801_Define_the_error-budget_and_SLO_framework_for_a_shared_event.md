---
type: "practice"
difficulty: "Expert"
---

# Define the error-budget and SLO framework for a shared event platform used by dozens of teams. How do you attribute a lag breach to the platform versus a tenant, and what does the budget policy trigger for each?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Breaking Schema Changes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Breaking_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define an error-budget and Service Level Objective (SLO) framework for a shared event platform used by dozens of teams, we need to establish clear metrics, understand potential sources of errors, and set up mechanisms to attribute them effectively. The primary goal is to ensure the platform meets its agreed service levels while providing visibility into performance degradation and enabling proactive resolution.

#### Triaging the Question

The core challenge here involves setting error-budgets and SLOs for a shared event platform. We need to define what constitutes an acceptable level of failure, monitor system performance, and allocate responsibilities among tenants and the platform itself. The key metrics include latency breaches, message delivery failures, and overall service availability.

#### Assumptions
Error Budget: A total budget of allowed errors or failures in a given period.
SLOs (Service Level Objectives): Agreed upon performance goals for the system.
Tenants: Individual teams using the event platform.
Lag Breaches: Instances where message processing exceeds acceptable latency thresholds.

#### First-Principles Solution
1. Define Error Budget and SLOs

First, define your error budget based on historical data and business requirements. For instance, if your system needs to be available for 99.95% of the time (SLO), an appropriate error budget could be up to 0.05% downtime.

Error Budget
Latency Breaches
Message Delivery Failures
System Downtime
2. Attribute Lag Breaches

To attribute lag breaches, we'll use a combination of monitoring tools and logs to identify the root cause. For each breach:

Platform Monitoring: Use metrics like processing latency, message backlog, and queue depth to pinpoint issues.
Tenant Logs: Review tenant-specific logs for any anomalies or known issues.
Monitoring System
Detect Latency Breaches
Log Analysis
Platform Issue Detected
Tenant Issue Detected
3. Budget Policy Triggers

When the error budget is breached, implement a policy that escalates issues based on severity:

Minor Breaches: Notify stakeholders via alerts and emails.
Moderate Breaches: Initiate an automated failover process or temporary rate limiting to prevent further degradation.
Severe Breaches: Trigger a full-scale incident response plan involving cross-functional teams.
4. Implementation Steps
Set Up Monitoring: Implement real-time monitoring for latency and message delivery metrics.
Define SLOs: Establish specific performance targets for the platform and tenants.
Implement Error Detection Mechanisms: Use APM tools to track breaches in real time.
Develop Attribution Logic: Create scripts or services that correlate tenant logs with system metrics.

#### Closing Summary

By defining an error budget, setting clear SLOs, and implementing a robust attribution framework, we can ensure the shared event platform meets its service commitments while providing visibility into performance issues. This approach helps in proactive issue resolution and maintains trust among teams using the platform.

#### Common Mistakes/Follow-Ups
Overlooking Tenant Impact: Ensure that tenant-specific issues are not incorrectly attributed to the platform.
Insufficient Monitoring: Relying solely on system-level metrics can miss critical tenant-specific issues.
Lack of Automation: Manual intervention in breach scenarios can delay resolution and lead to human errors.

**Comparison:**

```
Metric	Threshold (ms)	Action
Latency	10	Notify Stakeholders
	20	Initiate Failover
	30	Full Incident Response
```

**Code:**

```python
# Example Python code for monitoring latencies
import time

def monitor_latency():
    start_time = time.time()
    # Simulate processing a message
    process_message()
    end_time = time.time()

    latency = (end_time - start_time) * 1000  # in milliseconds
    if latency > MAX_LATENCY_THRESHOLD:
        log.warning(f"Latency breach detected: {latency} ms")
        trigger_error_budget_policy("latency")
```

**Example input / output:**

```text
Input:
- Latency breach detected at 15ms

Output:
- Alert sent to stakeholders via email
- Platform monitoring logs updated with event details
- Tenant logs reviewed for any known issues
```

**Data:**

```json
{
    "SLOs": {
        "system_availability": "99.95%",
        "message_delivery": "99.98%"
    },
    "Error_Budget": 0.05,
    "Breach_Triggers": [
        {"threshold": 10, "action": "Notify Stakeholders"},
        {"threshold": 20, "action": "Initiate Failover"},
        {"threshold": 30, "action": "Full Incident Response"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Latency Breach?
5. Log Breach
6. Monitor System Metrics
7. Analyze Tenant Logs
8. Check Next Metric

