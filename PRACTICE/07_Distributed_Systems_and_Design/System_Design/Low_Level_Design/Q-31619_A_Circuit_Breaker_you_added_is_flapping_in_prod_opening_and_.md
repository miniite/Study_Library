---
type: "practice"
difficulty: "Expert"
---

# A Circuit Breaker you added is flapping in prod, opening and closing every few seconds and making things worse; how do you debug the state transitions and tune the thresholds and half-open probe?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a circuit breaker that's flapping (opening and closing rapidly) in production, we need to understand its state machine behavior, identify potential misconfigurations or issues, and make necessary adjustments. Here’s how I would approach this problem:

Firstly, let's outline the core idea: A circuit breaker is designed to protect your system from overloading by failing fast when a service becomes unresponsive or slow, thus preventing cascading failures. However, in this case, it appears that the circuit breaker is misbehaving and causing more harm than good.

To debug the state transitions and tune the thresholds properly, we need to follow these steps:

Understand the State Machine: Circuit breakers typically have three states: Open, Half-Open, and Closed.
Review Configurations: Check if the trip threshold (number of failures before tripping), the reset period (time until it switches from Open back to Half-Open), and the half-open period (time between Half-Open and fully Closed) are correctly configured.
Examine Logs and Metrics: Inspect logs for failure patterns, and use monitoring tools to gather metrics on service response times and error rates.

Next, I would walk through each of these steps in detail:

1. Understanding the State Machine

A circuit breaker transitions between states based on these rules:

Open: When the trip threshold is exceeded.
Half-Open: After a period (reset period), allowing one request to pass through and observe if it succeeds or fails.
Closed: If the last transition from Half-Open was successful, it returns to Closed.
2. Review Configurations

Typically, the circuit breaker might be misconfigured due to overly aggressive thresholds or insufficient time to settle. Here are some common issues:

Trip Threshold Too Low: The system might be tripping too frequently because of minor service hiccups.
Reset Period Too Short: The reset period is set very short, leading to rapid state transitions.
Half-Open Period Inadequate: This ensures that the system can recover from failures before it starts making more requests.
3. Examine Logs and Metrics

To debug further:

Service Response Times: Look for any spikes in response times just before a failure event.
Error Rates: Identify if there’s an increase in errors right around the same time as a state transition.
Request Patterns: Analyze request patterns to see if there are specific types of requests that cause failures.
Step-by-Step Solution
Collect Detailed Metrics:
Service Requests
Response Time
Error Rate
Circuit Breaker StateTransitions
Analyze Logs and Metrics to identify patterns.
Adjust Thresholds: Based on insights from logs and metrics, adjust the trip threshold and reset period:
PYTHON
# Example in Python pseudo-code for adjusting circuit breaker thresholds
if service_response_times > 100ms and error_rate > 5%:
    set_trip_threshold(20)
    set_reset_period(60s)
    set_half_open_period(30s)
Test Changes: Deploy the adjusted configurations in a staging environment to observe behavior.
Monitor Production: After deployment, closely monitor production for any unexpected changes.

#### Closing Summary

By following these steps, we can effectively debug and tune the circuit breaker's state transitions to ensure it operates as intended without causing unnecessary disruptions.

#### Common Mistakes

Overlooking logs and metrics, which provide crucial insights.
Failing to test changes thoroughly in a staging environment.
Not adjusting configurations based on real-world data but sticking to hardcoded values.

**Comparison:**

```
Service Response Times	Error Rate
98ms	2%
102ms	3%
104ms	4%
105ms	6%
110ms	7%
```

**Code:**

```python
# Example Python pseudo-code for logging and checking circuit breaker state transitions
def check_state_transitions(service_response_times, error_rate):
    if service_response_times > 100ms and error_rate > 5%:
        set_trip_threshold(20)
        set_reset_period(60s)
        set_half_open_period(30s)
```

**Example input / output:**

```text
Input:
- Service response times: [98ms, 102ms, 104ms, 105ms, 110ms]
- Error rate: [2%, 3%, 4%, 6%, 7%]

Output:
- Circuit breaker state transitions observed as expected.
- Adjusted thresholds for better performance.
```

**Data:**

```json
{
  "service_response_times": [98, 102, 104, 105, 110],
  "error_rate": [2%, 3%, 4%, 6%, 7%]
}
```

**Flow:**

1. Service Requests
2. Response Time
3. Error Rate
4. Circuit Breaker StateTransitions

