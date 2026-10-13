---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design an automated rollback trigger tied to SLO burn rate rather than raw error thresholds, and why raw thresholds cause both false rollbacks and missed slow-burn regressions.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SLO Burn Rate Alerting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/SLO_Burn_Rate_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When designing a distributed system, ensuring the reliability of services is crucial. One common approach to achieving this is by implementing Service Level Objectives (SLOs), which define the desired performance levels. A robust rollback mechanism can help maintain service quality by automatically reverting to previous versions if the SLOs are not met. In this answer, I'll explain how to design an automated rollback trigger that uses SLO burn rate rather than raw error thresholds and discuss why using raw thresholds alone can lead to false rollbacks and missed slow-burn regressions.

#### Triage

The question is asking for a detailed explanation of designing an automated rollback mechanism based on Service Level Objective (SLO) burn rates instead of traditional error thresholds. The key points being probed are:

Understanding the difference between SLOs and raw error metrics.
Identifying the limitations of using raw error thresholds as sole criteria for rollbacks.
Explaining how to design a more comprehensive rollback mechanism based on SLO performance.

#### First Principles Solution
1. Define Service Level Objectives (SLOs)

Service Level Objectives are predefined targets that define acceptable levels of service quality. For example, an SLO might specify that the system should be available 99.9% of the time and respond within a certain latency threshold.

2. Monitor SLO Burn Rate

The burn rate is how quickly or slowly your actual performance metrics approach your SLO targets over time. This can provide more nuanced insights into service health than raw error counts because it captures trends and patterns that static thresholds might miss.

3. Set Up Automated Rollback Criteria Based on SLO Burn Rate

To design an automated rollback mechanism:

Calculate the burn rate for each SLO: For instance, if you have an availability SLO of 99.9%, track how closely your actual availability is to this target.
Define thresholds based on SLO burn rates: Instead of setting a hard error threshold (e.g., 100 errors), set thresholds that reflect the rate at which these errors accumulate relative to the SLO.

For example, if the burn rate for an availability SLO drops below 95% over a period of one hour, initiate a rollback. This approach allows you to detect and respond to slow degradation rather than waiting for catastrophic failures.

4. Implement Rollback Logic
Version Management: Maintain multiple versions of your service in a staging environment.
A/B Testing: Use canary deployments or A/B testing strategies to gradually roll out changes while monitoring their impact on SLOs.
Rollback Strategy: Define the rollback strategy, such as automatically reverting to the previous version if SLO burn rates fall below critical thresholds.

#### Closing Summary
and Common Mistakes/Follow-Ups

Using raw error thresholds alone can lead to false rollbacks because:

They might not capture the true state of service health.
They can trigger rollbacks for minor issues that don't significantly impact overall performance.

On the other hand, using SLO burn rates allows you to identify and address slow-burn regressions more effectively by providing a holistic view of how changes are affecting your system's performance over time. This approach aligns better with maintaining long-term service reliability and reduces the risk of false rollbacks.

#### Common Mistakes
Overlooking gradual degradation: Failing to recognize that subtle issues can slowly erode service performance.
Relying too heavily on error counts: Ignoring trends and patterns in SLO burn rates, leading to missed opportunities for proactive troubleshooting.
Inadequate monitoring: Not setting up comprehensive monitoring and alerting systems that track SLO performance.
Closing

By designing an automated rollback mechanism based on SLO burn rates, you can more effectively manage service reliability while reducing the risk of false rollbacks and missing slow-burn regressions. This approach provides a more nuanced and proactive way to ensure your distributed system maintains high-quality performance over time.

**Comparison:**

```
Metric	Target (%)	Actual (%)	Burn Rate (%)
Availability	99.9	98.5	-1.4
```

**Code:**

```python
# Example rollback logic based on SLO burn rate in Python

def calculate_burn_rate(actual, target):
    return (target - actual) / target

def trigger_rollback(burn_rate, threshold):
    if burn_rate < threshold:
        print("Rolling back to previous version.")
        # Implement actual rollback code here
```

**Example input / output:**

```text
Input:
- Availability SLO target: 99.9%
- Actual availability in last hour: 98.5%

Output:
- Burn rate for availability: -1.4% (indicating the system is degrading)
- Rollback triggered because burn rate < threshold
```

**Data:**

```json
{
  "availability_slo": {
    "target": 99.9,
    "actual": 98.5,
    "burn_rate": -1.4
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is SLO burn rate < threshold?
5. Rollback to previous version
6. Continue monitoring

