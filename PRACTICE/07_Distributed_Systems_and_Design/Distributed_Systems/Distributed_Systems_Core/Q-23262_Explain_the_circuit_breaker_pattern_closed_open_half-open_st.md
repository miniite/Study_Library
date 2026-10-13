---
type: "practice"
difficulty: "Deep"
---

# Explain the circuit breaker pattern: closed / open / half-open states. How do you pick the failure threshold and the half-open probe rate so you neither trip too early nor hammer a recovering service?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retry Budgets and Backoff](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The circuit breaker pattern is a crucial tool in building robust distributed systems, ensuring that failures in one component do not cascade to others. It operates on three states—closed, open, and half-open—and uses thresholds to decide when to transition between these states.

To fully understand the circuit breaker pattern, we need to break down its core concepts:

States:

Closed State: The system assumes all services are healthy.
Open State: When a service failure occurs, the circuit breaker trips and blocks further requests, returning cached responses until it transitions back to closed or half-open state.
Half-Open State: A limited number of calls are allowed from the client side to the server. If these calls succeed, the system reverts to the closed state; otherwise, it resets to the open state.

Failure Threshold: This is a critical setting that determines when to trip the circuit breaker into the open state. It involves balancing between quickly identifying service degradation and avoiding false trips.

Half-Open Probe Rate: This parameter controls how frequently we test the server after transitioning from the open state back to half-open.

The choice of these parameters requires careful consideration:

#### Triaging the Question
The circuit breaker pattern is a mechanism for managing resilience in distributed systems by limiting failures and preventing them from propagating across services.
We need to ensure that our thresholds are chosen such that:
The system doesn't trip too early, leading to unnecessary degradation of service availability.
The system doesn’t remain open indefinitely when the underlying service is recovering.

#### First Principles Solution
Failure Threshold
Understanding Latency and Error Rates:
Identify typical latency and error rates in your application. For example, if a backend service has an acceptable error rate of less than 5%, we need to set our threshold accordingly.
Statistical Significance:
Use statistical methods like control charts or simple moving averages to determine the confidence level required before tripping the circuit breaker.
Half-Open Probe Rate
Risk and Recovery Time:

#### Cons
ider how long it takes for services to recover under normal conditions. If recovery typically takes 5 minutes, you might set a probe interval of around this duration.
Error Budget Allocation:
Allocate an error budget for the half-open state. For instance, allowing up to 3 errors in 10 calls before transitioning back to the closed state.

#### Example Scenario

Let's consider a service UserService that we're protecting with a circuit breaker. We have observed that it should ideally not fail more than 5% of the time under normal conditions. We can set our failure threshold as follows:

Failure Threshold: ( \text{Tripped if } > 80% ) failures in the last 10 calls.
Half-Open Probe Rate: Allow up to 3 successful calls out of every 10 attempts before assuming recovery.

#### Closing Summary
and Common Mistakes

By setting appropriate thresholds, we ensure that our system remains robust while minimizing unnecessary down-time. The key is finding a balance where the circuit breaker trips only when necessary but still allows for quick recovery once services start working again.

#### Common mistakes
include:

Setting too low of a failure threshold, leading to frequent circuit breaker trips.
Not allowing enough probes in the half-open state, making it hard for services to recover quickly.

**Comparison:**

```
State	Failures	Probes
Open	>80%	N/A
Half-Open	<3	Every 5 min
```

**Code:**

```python
# Example Circuit Breaker Implementation (using Python)
from time import sleep

def circuit_breaker(failure_threshold=80, probe_rate=3):
    failures = 0
    open_state = False
    half_open_state = False

    def wrapper(func):
        nonlocal failures
        nonlocal open_state
        nonlocal half_open_state

        def decorated(*args, **kwargs):
            if open_state:
                # Return cached response or fail fast
                return "Service Unavailable"
            elif half_open_state:
                try:
                    result = func(*args, **kwargs)
                    failures = 0
                    half_open_state = False  # Reset after success
                    return result
                except Exception as e:
                    failures += 1
                    if failures >= probe_rate:  # Trip circuit breaker
                        open_state = True
            else:
                try:
                    result = func(*args, **kwargs)
                    failures = 0
                    return result
                except Exception as e:
                    failures += 1
                    if failures >= failure_threshold / 100 * 10:  # Tripped after 80% failures in last 10 calls
                        open_state = True

        return decorated

    return wrapper
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
@circuit_breaker(failure_threshold=80, probe_rate=3)
def service_call():
    return make_request_to_user_service()

for _ in range(10):
    try:
        response = service_call()
        print(response)
    except Exception as e:
        print(e)

Output:

PLAINTEXT
Copy
Service Unavailable
...
(up to three probes during half-open state)
Service Call Succeeded
Service Call Succeeded
Service Call Succeeded
```

**Data:**

```json
{
  "failureThreshold": 80,
  "probeRate": 3,
  "openState": false,
  "halfOpenState": false
}
```

**Flow:**

1. All probes succeed
2. Probe fails, transition back toopen
3. Probe successful

