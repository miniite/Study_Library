---
type: "practice"
difficulty: "Intermediate"
---

# How do you design gateway behavior when an upstream returns Retry-After on a 429/503; should the gateway honor it, transform it, or hide it from clients, and how does that interact with your own retry policy?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Timeouts Retries and Backoff](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Timeouts_Retries_and_Backoff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When designing an API Gateway to handle upstream responses like Retry-After on 429 (Too Many Requests) or 503 (Service Unavailable) status codes, you need to consider how to manage retries, maintain client experience, and align with your own retry policy. The key decision points are whether to honor the Retry-After, transform it, or ignore it entirely.

#### Triage

The question is essentially asking for a design strategy that balances performance, reliability, and user experience. Here, we need to understand:

Honor Retry-After: If you respect the Retry-After value, your gateway will introduce delays between retries based on the specified duration.
Transform Retry-After: You could modify the delay to fit better with your own retry policy or apply additional logic.
Hide Retry-After: You might choose to ignore this header and implement a consistent retry policy across all API calls.

#### Assumptions
The upstream service returns 429 or 503 status codes when it is overloaded or temporarily unavailable.
Your API Gateway needs to handle these responses in a way that provides a good user experience while maintaining the integrity of your application's retry logic.
Step-by-Step Solution
Option 1: Honor Retry-After

#### Pros

Adhere strictly to the upstream service’s policy, ensuring no unnecessary load is placed on it.
Maintain transparency and trust with third-party services.

#### Cons

May lead to inconsistent behavior depending on the upstream’s implementation.
Client applications might not be aware of the specific delays imposed by the upstream.
API Gateway
Upstream Service
429/503 + Retry-After
Gateway HONORS Retry-After
Delay client request forspecified duration
Option 2: Transform Retry-After

#### Pros

You can apply a consistent retry policy across multiple upstream services.
Can optimize the delay to better fit your application’s needs.

#### Cons

May not always align with the intended behavior of the upstream service.
Adds complexity in managing and maintaining the transformation logic.
API Gateway
Upstream Service
429/503 + Retry-After
Gateway TRANSFORMSRetry-After
Apply custom delay logic
Option 3: Hide Retry-After

#### Pros

Simplifies the client’s implementation by abstracting away the specific retry delays.
Ensures consistent behavior regardless of upstream policy changes.

#### Cons

May lead to more frequent retries, potentially overwhelming the upstream service.
Loss of transparency and potential misinterpretation of failure modes.
API Gateway
Upstream Service
429/503 + Retry-After
Gateway HIDES Retry-After
Implement own retry policywithout client awareness
Interacting with Your Own Retry Policy

Regardless of the chosen option, your gateway should align its behavior with your overall retry strategy. For instance:

Backoff Strategy: Implement a backoff algorithm that respects the Retry-After but also scales according to your application’s needs.
Circuit Breaker: Integrate circuit breaker patterns to prevent overwhelming the upstream during prolonged failures.

#### Closing Summary

Designing how an API Gateway handles Retry-After involves balancing trust, consistency, and control. Honoring it aligns with upstream policies but can be inconsistent; transforming it offers flexibility at the cost of added complexity; hiding it simplifies client interaction but may impact performance.

#### Common Mistakes/Follow-Ups
Ignoring Retry Logic: Failing to implement any retry logic at all can lead to frequent client errors.
Over-reliance on Retry-After: Blindly respecting this header without context can cause delays that are too long or too short.
Not Considering Circuit Breakers: Failing to incorporate circuit breaker patterns can result in a flood of requests during an outage.

**Comparison:**

```
Retry Policy	Initial Delay (Seconds)	Backoff Factor	Maximum Retries
Exponential	1	2	5
```

**Code:**

```python
# Example of implementing a retry policy with backoff logic
def handle_retry_after(retry_after, max_retries=5):
    current_delay = 1
    for attempt in range(max_retries):
        if attempt > 0:
            time.sleep(current_delay)
        # Make API call here
        response = make_api_call()
        if response.status_code != 429 and response.status_code != 503:
            return response

        current_delay *= 2  # Exponential backoff
```

**Example input / output:**

```text
Input:
- API Gateway receives a 429 response with Retry-After header set to 60 seconds from an upstream service.

Output:
- Gateway implements exponential backoff starting at 1 second, doubling each retry until the maximum of 5 retries is reached or a successful response is received.
```

**Data:**

```json
{
    "retry_policy": {
        "max_retries": 5,
        "initial_delay_seconds": 1,
        "backoff_factor": 2,
        "jitter_percentage": 0.2
    }
}
```

**Flow:**

1. Yes
2. No
3. API Gateway
4. Upstream Service
5. 429/503 + Retry-After
6. Gateway HONORS Retry-After
7. Delay client request forspecified duration
8. API Call
9. Check status_code != 429, 503
10. Make API Call
11. Increase delay and retry

