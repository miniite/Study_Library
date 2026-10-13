---
type: "practice"
difficulty: "Intermediate"
---

# An engineer set an autoscaling target-tracking policy on 50% CPU but the fleet oscillates between scale-out and scale-in every few minutes. What tuning of cooldowns and target value would you apply and why?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Autoscaling Metrics](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where an autoscaling policy configured to target 50% CPU utilization is causing frequent scaling events, we need to fine-tune the cooldown period and possibly adjust the target value. The primary goal is to reduce the oscillation between scale-out and scale-in actions by ensuring that the system has enough time to stabilize its load before making further adjustments.

Firstly, let's clarify what the question is probing:

We need to understand how AWS Auto Scaling policies work.
We should consider the trade-offs between responsiveness and stability in scaling strategies.
The autoscaling cooldown period and target value are key parameters that affect the behavior of the fleet.

#### Assumptions
for this scenario include:

The application has a variable load pattern, leading to frequent CPU fluctuations.
The cooldown time is set too short for the system to stabilize after each scaling event.
The target utilization (50%) may be too aggressive or not aligned with the actual workload profile.

Step-by-step reasoning from first principles:

Cooldown Period:

The cooldown period defines the minimum wait time before another scaling activity can occur. A shorter cooldown leads to more frequent but potentially less accurate adjustments, while a longer cooldown reduces noise and allows for better long-term trends.
Given that the current behavior is oscillating frequently every few minutes, we should increase the cooldown period. A typical starting point might be increasing it from 1 minute (the default) to at least 3-5 minutes.

Target Value Adjustment:

The target value determines how often and by what magnitude the scaling group will adjust its capacity in response to changes in CPU utilization.
If the current target is set to 50%, we might need to consider if this is too aggressive. A higher target, such as 60% or even 70%, could help avoid frequent adjustments while still maintaining responsiveness.

Target Tracking Policy:

Ensure that the Target Tracking Policy is properly configured with the correct metric (CPU Utilization) and its dimensions.
Verify that the policy's predefined actions are appropriate for the scaling group size, ensuring that it can handle both scale-out and scale-in operations effectively without causing excessive oscillations.

Monitoring and Validation:

After adjusting the cooldown period and target value, closely monitor the system’s behavior over a period to confirm that the changes have reduced oscillation.

#### Cons
ider setting up CloudWatch alarms for threshold breaches to proactively detect issues before they become critical.

#### Closing Summary
In summary, tuning the autoscaling policy involves increasing the cooldown period to 3-5 minutes and adjusting the target value from 50% to a higher percentage like 60%-70%. This helps stabilize the system by reducing frequent adjustments based on short-term fluctuations. It is crucial to monitor the system post-tuning to validate that these changes have improved stability without compromising responsiveness.

#### Common Mistakes/Follow-Ups

Ignoring the cooldown period, which can lead to unnecessary scaling events.
Not considering the actual workload profile when setting target values.
Failing to properly configure and test the new settings before deployment.

