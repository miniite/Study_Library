---
type: "practice"
difficulty: "Deep"
---

# An AI-suggested circuit breaker never transitions out of open because it lacks a half-open probe. How do you spot the missing state and prompt for a correct three-state breaker with tests?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing AI Code for Performance Risks](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
The core issue here is that an AI-assisted circuit breaker, which should ideally transition through "open," "half-open," and "closed" states, is stuck in a perpetual open state due to missing functionality. Specifically, the lack of a "half-open" probe prevents the system from attempting to re-enable itself. To address this, we need to add this state, implement the necessary logic for probes, and ensure robust testing.

#### Triage
The question is asking us to identify why an AI-suggested circuit breaker implementation fails to recover properly and how to correct it by adding a "half-open" state with associated probe logic. We will assume that the existing "open" and "closed" states are functioning correctly but fail to handle network failures or retries appropriately.

#### First-Principles Solution

Define the States: Introduce a new HalfOpen state between Open and Closed.
Implement Probes: Add logic for a half-open probe that checks if the service is recoverable.
Transition Logic: Ensure the system transitions from Open to HalfOpen when it starts receiving success signals, then to Closed on successful probes.

Let's break this down step-by-step:

Define the States:

Open: The circuit breaker is in an open state and no requests are processed.
HalfOpen: The circuit breaker allows a limited number of requests (probes) before deciding to fully close or open.
Closed: The circuit breaker processes all incoming requests as usual.

Implement Probes:

Define the halfOpenProbe method which will send a small number of probes to determine if the service is stable again.
If successful, transition to Closed.

Transition Logic:

When transitioning from Open to HalfOpen, set up a timer for retries based on backoff strategy (e.g., exponential).
On each retry, send probes and update state accordingly.

#### Closing Summary
By introducing the HalfOpen state and implementing robust probe logic, we can ensure that the circuit breaker transitions correctly, avoiding an endless open state. Testing will be crucial to validate these changes thoroughly.

#### Common Mistakes/Follow-Ups

Not fully understanding the difference between open, half-open, and closed.
Failing to implement proper backoff and probe logic.
Ignoring edge cases such as network outages during probes or invalid responses from services.

**Code:**

```python
class CircuitBreaker:
    def __init__(self, open_state_timeout=5):
        self.state = 'Closed'
        self.open_state_timer = 0
        self.retries_left = 3  # Number of retries before fully opening

    def halfOpenProbe(self):
        if self.state == 'HalfOpen':
            success = self.send_probe()  # Simulate sending a probe request
            if success:
                self.state = 'Closed'
                self.open_state_timer = 0
                self.retries_left = 3
            else:
                self.state = 'Open'

    def send_probe(self):
        # Placeholder for actual network call and logic to check if the service is stable
        return True

    def handle_request(self, request):
        if self.state == 'Open':
            print("Request rejected due to open state")
            return None

        elif self.state == 'HalfOpen':
            self.halfOpenProbe()
            return self.send_probe()

        else:  # State is Closed
            response = self.send_real_service(request)
            if response.success:
                self.open_state_timer += 1
                if self.open_state_timer >= open_state_timeout:
                    self.state = 'Closed'
                else:
                    print("Request accepted and processed")
            else:
                self.open_state_timer = 0
                self.state = 'Open'
                print("Request rejected due to failure")
```

**Flow:**

1. Closed
2. Handle Request
3. Send Real Service
4. If Success
5. Increment Timer
6. Timeout Check
7. Close
8. If Failure
9. Reset Timer
10. Open State
11. HalfOpen
12. Handle Request
13. Send Probe
14. Probe Success
15. Close State
16. Probe Failure
17. Open State

