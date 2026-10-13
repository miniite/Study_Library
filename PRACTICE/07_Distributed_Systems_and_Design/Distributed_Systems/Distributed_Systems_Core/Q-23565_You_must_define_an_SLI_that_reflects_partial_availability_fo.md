---
type: "practice"
difficulty: "Deep"
---

# You must define an SLI that reflects partial availability for a system where 3 of 30 features can be down while users still 'succeed.' Explain why uptime% is inadequate and how you'd compose per-journey success rate, tail latency, and partial degradation into a single defensible SLI.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ SLO Burn Rate Alerting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/SLO_Burn_Rate_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of defining an SLI (Service Level Indicator) that reflects partial availability in a system where 3 out of 30 features can be down, yet users still "succeed," we need to go beyond simple uptime metrics like percentage uptime. Uptime% alone does not capture the nuances of journey success and tail latency, which are crucial for understanding the overall user experience.

#### Triaging the Question

The key aspects the interviewer is probing include:

Understanding Uptime Limitations: Recognizing why uptime% might be inadequate.
Defining Success Metrics: Composing a SLI that considers per-journey success rate, tail latency, and partial degradation.
Technical Depth: Demonstrating a deep understanding of distributed systems and their complexities.
Step-by-Step Solution From First Principles
1. Understanding Uptime Limitations

Uptime percentage is typically defined as the ratio of time when the system is available to the total monitoring period. However, this metric doesn't account for partial availability scenarios where some features can be down but users still succeed in their overall journeys.

For example, imagine a system with 30 different features; if three of these features are down but the user's primary journey (which depends on only 27 features) still completes successfully, uptime% would not reflect this success.

2. Defining Success Metrics

To address this, we need to define an SLI that considers:

Per-journey Success Rate: The probability that a given user journey will succeed.
Tail Latency: The performance of the system during peak loads or when dealing with high volumes of requests.
Partial Degradation: The extent to which some features can be down without impacting the overall success rate.
3. Composing the SLI

A possible SLI could be a weighted composite metric that considers these factors: [ \text{SLI} = w_1 \times (\text{Per-journey Success Rate}) + w_2 \times (\text{Tail Latency}) - w_3 \times (\text{Partial Degradation}) ]

Where (w_1), (w_2), and (w_3) are weights that need to be determined based on the system's criticality and user expectations.

4. Example Calculation

Let’s break down an example:

Per-journey Success Rate: Suppose it is 95%.
Tail Latency: Assume the tail latency in milliseconds (ms) during peak loads is 20 ms.
Partial Degradation: If three features can be down without affecting journey success, we might assign a penalty of -10%.

Using weights (w_1 = 0.5), (w_2 = 0.3), and (w_3 = 0.2): [ \text{SLI} = 0.5 \times 95% + 0.3 \times (20 , \text{ms}) - 0.2 \times (-10%) ] [ \text{SLI} = 47.5% + 6 , \text{ms} + 2% ] [ \text{SLI} = 55.5% ]

#### Closing Summary
and Common Mistakes

By defining the SLI as a composite metric that includes per-journey success rate, tail latency, and partial degradation, we can capture a more nuanced understanding of system availability and performance. The common mistakes to avoid include:

Over-relying on simple uptime metrics.
Ignoring user journeys and their successes or failures.
Not accounting for performance during peak loads.
Failing to consider the impact of partial feature degradation.
Common Follow-Ups
How would you measure per-journey success rate?
What are some trade-offs between these different metrics?
How would you adjust the weights (w_1), (w_2), and (w_3) in a real-world scenario?

**Comparison:**

| Metric               | Weight | Value   |
|----------------------|--------|---------|
| Per-journey Success Rate | 0.5    | 95%     |
| Tail Latency         | 0.3    | 20 ms   |
| Partial Degradation  | -0.2   | -10%    |

**Code:**

```python
# Example Python code to calculate the SLI
def calculate_sli(per_journey_success_rate, tail_latency_ms, partial_degradation):
    w1 = 0.5
    w2 = 0.3
    w3 = -0.2

    sli = (w1 * per_journey_success_rate) + (w2 * tail_latency_ms) + (w3 * partial_degradation)
    return sli

# Example values
per_journey_success_rate = 95 / 100
tail_latency_ms = 20
partial_degradation = -10 / 100

sli = calculate_sli(per_journey_success_rate, tail_latency_ms, partial_degradation)
print(f"SLI: {sli * 100:.2f}%")
```

**Example input / output:**

```text
Input:
- per_journey_success_rate = 95%
- tail_latency_ms = 20 ms
- partial_degradation = -10%

Output:
- SLI = (0.5 * 95%) + (0.3 * 20 ms) - (0.2 * -10%)
      = 47.5% + 6 ms + 2%
      = 55.5%

Calculation Result: SLI = 55.5%
```

**Data:**

```json
{
  "per_journey_success_rate": 95,
  "tail_latency_ms": 20,
  "partial_degradation": -10
}
```

**Flow:**

1. No
2. Define SLI
3. Is Uptime% Sufficient?
4. Consider Per-journey SuccessRate, Tail Latency, and PartialDegradation
5. Compose Composite Metric
6. Calculate SLI
7. Monitor and Adjust Weights

