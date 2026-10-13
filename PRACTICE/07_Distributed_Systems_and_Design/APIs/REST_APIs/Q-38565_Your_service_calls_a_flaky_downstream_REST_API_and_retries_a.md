---
type: "practice"
difficulty: "Expert"
---

# Your service calls a flaky downstream REST API and retries are amplifying its outage into your own. How do you distinguish a retry storm from real load, and what combination of circuit breaker, backoff, and load shedding do you apply?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Long-running Operations](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Long-running_Operations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where your service is calling a flaky downstream REST API and retries are amplifying outages into your own system, we need to distinguish between retry storms (unnecessary retries) and real load. The solution involves combining circuit breakers, backoff strategies, and potentially load shedding techniques.

First, let's triage the question: The interviewer is probing how you can effectively manage retries in a distributed system to prevent amplifying downstream issues into your own service. They are likely concerned with robustness, performance, and reliability of the system under high load conditions. Assumptions include:

The flaky API has intermittent failures.
Retries are needed but must be managed carefully.

From first principles, we will design a strategy that includes:

Circuit Breaker: To quickly fail requests to the downstream service once it starts failing and stop retrying until it recovers.
Backoff Strategy: To gradually increase the delay between retries to avoid overwhelming the flaky API and other dependent services.
Load Shedding: To limit the number of concurrent requests from your service to the downstream API.

Let's break this down step by step:

Step 1: Implement a Circuit Breaker

A circuit breaker detects when the downstream service is failing at an unacceptable rate and switches to a fallback mechanism (like returning cached data or an error message). This prevents excessive retries and protects the downstream system from overload.

Yes
No
Service Calls Downstream API
API Fails?
Circuit Breaker Trip
Return Error/Fallback Data
Retry After Backoff
Step 2: Configure Backoff Strategy

Backoff strategies control the rate at which retries are performed. Common backoff algorithms include exponential backoff, where each retry doubles the delay.

PYTHON
def exponential_backoff(max_retries=5):
    for i in range(1, max_retries + 1):
        time.sleep(i * 2)  # Sleep for 2 seconds after first attempt, 4s after second, etc.
        try:
            response = call_downstream_api()
            if response.status_code < 500:  # Assuming successful if status code is not an error
                return response
        except Exception as e:
            continue
    raise ServiceUnavailableException("Failed to reach downstream API")
Step 3: Implement Load Shedding

Load shedding limits the number of concurrent requests to the flaky API. This can be achieved by using a rate limiter or a semaphore.

Yes
No
Service Calls Downstream API
Rate Limiter Allows?
Send Request
Wait and Retry Later
Step 4: Monitor and Adjust

Finally, continuously monitor the health of your service and the downstream API to adjust parameters as necessary. This could involve adjusting the circuit breaker thresholds or changing backoff durations based on historical data.

PYTHON
def monitor_and_adjust(api_health_metrics):
    if api_health_metrics.fails_per_second > threshold:
        circuit_breaker_trip(api_health_metrics)

#### Closing Summary
and Common Mistakes/Follow-Ups

By combining these strategies, you can effectively manage retries and prevent amplifying downstream outages. Common mistakes include not implementing a proper fallback mechanism or ignoring backoff, leading to continuous retries that overwhelm the system. Other follow-ups might involve discussing specific implementations, trade-offs between different strategies, or exploring advanced techniques like smart circuit breakers with machine learning for better accuracy.

