---
type: "practice"
difficulty: "Deep"
---

# Design a Circuit Breaker class. What are its states and what triggers each transition?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a Circuit Breaker class, we need to ensure it can handle failures gracefully while providing reliable service. The circuit breaker pattern helps in managing service disruptions by switching off a failing service until it is deemed safe again. This design involves multiple states and transitions that allow for controlled degradation of the system.

#### Triaging the Question

The question asks us to design a Circuit Breaker class, which means we need to define its internal state machine and the conditions under which each state transition occurs. We should consider common states like Open, Half-Open, and Closed along with their transitions. The interviewer is likely probing our understanding of fault tolerance, resilience patterns, and how to implement these concepts in a robust manner.

#### First Principles Solution

A Circuit Breaker class typically has three main states: Open, Half-Open, and Closed. Each state represents different behavior regarding the service calls:

Closed: The circuit breaker is fully open and allows all requests to pass through.
Open: All incoming requests are rejected, and no further attempts to reset or half-open the breaker are allowed until a predefined timeout period has elapsed.
Half-Open: A single request is allowed to test if the service is stable. If successful, the circuit breaker transitions back to Closed; otherwise, it returns to Open.
Detailed State Transitions
From Closed to Open:
Trigger: Service failure threshold exceeded (e.g., 5 consecutive failures).
From Open to Half-Open:
Trigger: Timeout period for resetting has elapsed.
From Half-Open to Closed:
Trigger: Successful request after half-open state is entered.

#### Implementation
PYTHON
class CircuitBreaker:
    class State(Enum):
        CLOSED = 1
        OPEN = 2
        HALF_OPEN = 3

    def __init__(self, failure_threshold=5, reset_timeout=60):
        self.state = CircuitBreaker.State.CLOSED
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.last_failure_time = None
        self.attempt_count = 0

    @property
    def is_closed(self):
        return self.state == CircuitBreaker.State.CLOSED

    @property
    def is_open(self):
        return self.state == CircuitBreaker.State.OPEN

    @property
    def is_half_open(self):
        return self.state == CircuitBreaker.State.HALF_OPEN

    def attempt_call(self, callable_function):
        if not self.is_closed and self.attempt_count < self.failure_threshold:
            try:
                result = callable_function()
                self.reset_state()
                return result
            except Exception as e:
                self.fail_call(e)
                return None
        else:
            # In Open or Half-Open state, reject all calls
            print("Circuit Breaker open: rejecting request.")
            return None

    def fail_call(self, exception):
        if not self.is_half_open and self.attempt_count < self.failure_threshold - 1:
            self.attempt_count += 1
        elif self.is_half_open:
            self.state = CircuitBreaker.State.OPEN
            self.last_failure_time = time.time()

    def reset_state(self):
        if self.is_open or self.is_half_open:
            self.attempt_count = 0
            self.state = CircuitBreaker.State.HALF_OPEN

#### Closing Summary
and Common Mistakes/Follow-Ups

The Circuit Breaker class effectively manages the states of a service call to ensure reliability. The key transitions and states provide mechanisms for fault tolerance without overwhelming system resources during failures.

#### Common Mistakes

Forgetting to reset the state properly.
Setting incorrect thresholds or timeouts that can lead to prolonged downtimes.
Not handling edge cases where calls might be made while in an open or half-open state.

#### Follow-Ups

The interviewer may probe deeper into:

How to handle distributed systems and retries.

#### Trade-offs
between circuit breaker states and system performance.
Integration with other resilience patterns like bulkheads and rate limiters.
Real-world examples of when a Circuit Breaker would be useful.

