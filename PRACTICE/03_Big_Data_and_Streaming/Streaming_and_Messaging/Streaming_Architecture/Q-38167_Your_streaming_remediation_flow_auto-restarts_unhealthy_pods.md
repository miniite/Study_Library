---
type: "practice"
difficulty: "Deep"
---

# Your streaming remediation flow auto-restarts unhealthy pods based on a stream of health events. How do you add a circuit breaker so a bad event storm doesn't trigger a restart loop across the fleet?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Duplicates During Rebalance](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Duplicates_During_Rebalance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of preventing a restart loop in a real-time streaming remediation flow, we need to implement a circuit breaker mechanism that can detect and mitigate rapid-fire health event storms. At an AIOps Architect/Principal level, this involves understanding both the streaming architecture and the principles of robust fault management.

The core idea is to use a circuit breaker pattern to temporarily halt the restart process when a high volume of unhealthy events occurs within a short period, thus avoiding cascading failures across the pod fleet.

#### Triaging the Question
Objective: Implement a mechanism that can prevent a restart loop triggered by frequent health event storms.
Probing Areas:
Understanding of streaming architectures and real-time processing.
Knowledge of fault tolerance and resilience patterns like circuit breakers.

#### Trade-offs
between availability, latency, and cost.

#### Assumptions
The system already has a mechanism to detect unhealthy pods from health events.
Each unhealthy event triggers a pod restart request.
We need to ensure that the system can handle occasional transient failures without triggering an endless loop of restarts.

#### First-Principles Solution
1. Define Event Storm Threshold
Threshold Definition: Establish a threshold for the number and frequency of health events within a sliding window (e.g., 50 events in 2 minutes).

#### Implementation
Use a counter that increments with each unhealthy event, resetting after the window expires.
PYTHON
event_counter = 0
SLIDING_WINDOW_DURATION = 120  # seconds

def on_health_event(is_unhealthy):
    global event_counter
    if is_unhealthy:
        event_counter += 1
    else:
        event_counter -= 1

    if event_counter > THRESHOLD and time.time() - last_reset_time < SLIDING_WINDOW_DURATION:
        # Circuit breaker open: no restarts until the window expires
        return False
    else:
        # Reset counter or allow restarts
        reset_counter()
        return True

def reset_counter():
    global event_counter, last_reset_time
    event_counter = 0
    last_reset_time = time.time()

2. Implement Circuit Breaker Logic
Circuit State: Maintain a state machine to track the circuit breaker's status (open/closed).
Break Open Condition: If the threshold is exceeded within the sliding window, set the circuit breaker to open.
Break Close Condition: After the window expires or after sufficient time without new events, reset the circuit breaker.
PYTHON
CIRCUIT_OPEN = "OPEN"
CIRCUIT_CLOSED = "CLOSED"

circuit_state = CIRCUIT_CLOSED

def handle_circuit_breaker(event):
    global circuit_state

    if not on_health_event(event.is_unhealthy):
        # Circuit breaker is open: no restarts
        return False

    # Circuit breaker closed: allow restarts
    if circuit_state == CIRCUIT_OPEN:
        circuit_state = CIRCUIT_CLOSED
    return True
3. Evaluate and Optimize
Tuning Parameters: The threshold and sliding window duration should be tuned based on historical data to strike a balance between responsiveness and robustness.
Logging and Monitoring: Implement logging to track when the circuit breaker opens and closes, aiding in post-mortem analysis.

#### Closing Summary

In summary, adding a circuit breaker involves defining thresholds for event storms within a sliding window. The system monitors health events, temporarily halts restarts during high volumes of unhealthy events, and only allows restarts once the storm subsides. This approach ensures that transient failures do not cause cascading outages.

#### Common Mistakes/Follow-Ups
Incorrect Threshold Tuning: Setting thresholds too low can lead to frequent false positives, while setting them too high may miss critical failure points.
Insufficient Monitoring: Lack of logging and monitoring can make it hard to diagnose issues when the circuit breaker triggers or fails to close.

