---
type: "practice"
difficulty: "Deep"
---

# How do you design a circuit breaker so it opens on real failures but does not trip on a brief latency spike, and how do you tune half-open recovery?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Testing in Production](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Testing_in_Production.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A well-designed circuit breaker is crucial in managing service disruptions while avoiding false positives from transient issues like latency spikes. To achieve this, we implement a sliding window mechanism to distinguish between real failures and temporary issues, and configure the half-open state carefully to prevent unnecessary retries.

#### Triaging the Question

The question asks about designing a robust circuit breaker that can differentiate between true failures and brief latency spikes. It also inquires about tuning the half-open state, which involves managing the transition back from open to half-open after a period of non-failure conditions is observed.

Core Concepts
Circuit Breaker Pattern: A pattern used for preventing cascading failures by temporarily stopping requests to a failing service.
Sliding Window Mechanism: Used to track recent failure rates and make decisions based on the window's state.
Half-Open State: Allows some requests through after the circuit breaker trips, enabling gradual recovery.

#### First Principles Solution

Define Failure Metrics:

Identify what constitutes a "failure". For example, 5xx HTTP status codes or response times exceeding a certain threshold.

Sliding Window Mechanism:

Window Size: Define the window size (e.g., 30 seconds). This is the duration over which we track failures.
Failure Threshold: Determine the number of consecutive failures required to trip the circuit breaker (e.g., 5 failures within the window).

Monitoring and Logging:

Implement logging for all requests to ensure that the failure metric tracking is accurate.

Circuit Breaker States:

Closed State: Normal operation.
Open State: All incoming requests are immediately failed, stopping service degradation.
Half-Open State: A limited number of requests (e.g., 10) are allowed to pass through the circuit breaker.
Detailed Steps
Step 1: Define Failure Metrics
PYTHON
def is_request_successful(response):
    return response.status_code < 500 and response.elapsed.total_seconds() <= MAX_RESPONSE_TIME
Step 2: Sliding Window Mechanism Implementation
PYTHON
from collections import deque

class CircuitBreaker:
    def __init__(self, max_failures=5, window_size=30):
        self.failures = deque(maxlen=window_size)

    def record_response(self, response):
        if not is_request_successful(response):
            self.failures.append(True)  # Record a failure
        else:
            self.failures.clear()  # Clear failures on success

    def should_open(self):
        return len(self.failures) >= max_failures and all(failure for failure in self.failures)

# Example Usage
breaker = CircuitBreaker()
breaker.record_response(some_request)
if breaker.should_open():
    print("Circuit Breaker Open")
Step 3: Half-Open State Management
PYTHON
class CircuitBreaker:
    def __init__(self, max_failures=5, window_size=30):
        self.failures = deque(maxlen=window_size)
        self.opened = False

    def record_response(self, response):
        if not is_request_successful(response):
            self.failures.append(True)  # Record a failure
            if len(self.failures) >= max_failures and all(failure for failure in self.failures):
                self.open()
        else:
            self.close()  # Clear failures on success

    def open(self):
        self.opened = True

    def close(self):
        self.opened = False

    def half_open(self):
        if not self.opened and len(self.failures) == 0:  # Half-open after a successful response
            self.opened = True

    def should_open(self):
        return self.opened and all(failure for failure in self.failures)

    def handle_request(self, request):
        if self.should_open():
            return "Circuit Breaker Open"
        elif not self.opened:
            self.half_open()
            response = try_request(request)  # Attempt a request
            if is_request_successful(response):
                self.close()  # Transition to closed state on success
                return response
            else:
                return "Request Failed"
        else:
            return "Circuit Breaker Open"

#### Closing Summary
and Common Mistakes
Common Mistake: Setting the failure threshold too low can lead to frequent circuit breaker trips, while setting it too high can fail to protect against real failures.
Tuning Tip: Start with a conservative window size and increase as needed. Monitor production traffic to ensure that false positives are minimized.
Final Thoughts

Designing an effective circuit breaker involves balancing between protecting the system from cascading failures and avoiding unnecessary trips due to brief latency spikes. By implementing a sliding window mechanism and carefully tuning the half-open state, we can create a robust solution that meets these requirements.

**Comparison:**

```
Circuit Breaker State	Description
Closed	Normal operation. Requests are handled as usual.
Open	All requests fail immediately to prevent further service degradation.
Half-Open	A limited number of requests (10) pass through the breaker for testing.
```

**Code:**

```python
from collections import deque

class CircuitBreaker:
    def __init__(self, max_failures=5, window_size=30):
        self.failures = deque(maxlen=window_size)
        self.opened = False

    def record_response(self, response):
        if not is_request_successful(response):
            self.failures.append(True)  # Record a failure
            if len(self.failures) >= max_failures and all(failure for failure in self.failures):
                self.open()
        else:
            self.close()  # Clear failures on success

    def open(self):
        self.opened = True

    def close(self):
        self.opened = False

    def half_open(self):
        if not self.opened and len(self.failures) == 0:  # Half-open after a successful response
            self.opened = True

    def should_open(self):
        return self.opened and all(failure for failure in self.failures)

    def handle_request(self, request):
        if self.should_open():
            return "Circuit Breaker Open"
        elif not self.opened:
            self.half_open()
            response = try_request(request)  # Attempt a request
            if is_request_successful(response):
                self.close()  # Transition to closed state on success
                return response
            else:
                return "Request Failed"
        else:
            return "Circuit Breaker Open"

def is_request_successful(response):
    return response.status_code < 500 and response.elapsed.total_seconds() <= MAX_RESPONSE_TIME

breaker = CircuitBreaker()
breaker.record_response(some_request)
print(breaker.should_open())
```

**Example input / output:**

```text
breaker = CircuitBreaker()
breaker.record_response(request1)  # Simulated request, success
breaker.record_response(request2)  # Simulated request, success
breaker.record_response(request3)  # Simulated request, success
breaker.record_response(request4)  # Simulated request, success
breaker.record_response(request5)  # Simulated request, failure
breaker.record_response(request6)  # Simulated request, failure
breaker.record_response(request7)  # Simulated request, failure

print(breaker.should_open())  # True
breaker.handle_request(request8)  # Half-open state
breaker.handle_request(request9)  # Closed state after successful request
```

**Data:**

```yaml
breakers:
- max_failures: 5
- window_size: 30
- failure_threshold: 5
```

**Flow:**

1. Record Failure
2. No Failures
3. Handle Request
4. Success
5. Failure
6. Closed
7. Open
8. Close
9. Half-Open

