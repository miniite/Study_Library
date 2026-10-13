---
type: "practice"
difficulty: "Deep"
---

# You need admission control on the hot path to shed non-critical work during overload. Explain how you'd keep the shedding decision cheap and avoid shedding the health-check and control-plane traffic you need to recover.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Slow Dependencies and Load Shedding](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need a robust admission controller that can handle load spikes efficiently while ensuring critical control plane and health check traffic remains unaffected. The core idea is to implement a permissive mechanism that only sheds non-essential workloads during peak times without impacting essential operations.

First, let’s break down what the interviewer is probing: understanding of distributed system principles, mechanisms for load management, and strategies to ensure service availability under high load conditions.

We will assume several key points:

The application has a well-defined hot path where non-critical workloads are generated.
Control plane traffic includes health checks and critical operations that must remain unimpeded.
The system can handle periodic or occasional spikes in traffic without significant degradation of service quality.
Step-by-Step Solution
1. Understanding the Problem
Objective: Implement admission control to shed non-critical workloads during load spikes.

#### Cons
traints: Ensure that essential control plane and health check traffic are not affected.
2. Designing the Admission Control Mechanism

To achieve this, we can leverage a combination of rate limiting and dynamic queuing mechanisms:

Rate Limiting:

Use a sliding window approach to track incoming requests over time.
Set thresholds based on historical data or real-time monitoring metrics (e.g., CPU usage, memory consumption).
If the threshold is exceeded, start rejecting non-critical workloads.
PYTHON
def is_overloaded():
    # Check if current load exceeds predefined limits
    return get_current_load() > MAX_LOAD_THRESHOLD

def process_request(request):
    if not is_overloaded():
        handle_critical_traffic()
        handle_non_critical_traffic(request)
    else:
        log("Overload detected, rejecting non-critical request")

Dynamic Queuing:

Implement a separate queue for critical traffic and another for non-critical workloads.
Ensure that the critical queue has higher priority and is serviced first.
3. Ensuring Control Plane Traffic Is Not Impacted

Control plane traffic such as health checks should always be prioritized:

Health Check Handling:

Define a separate health check endpoint with a low rate limit (e.g., every few minutes).
Ensure this endpoint uses a different queue or direct connection to the application's critical path.
PYTHON
def handle_health_check():
    # Low rate limit and high priority for health checks
    if is_health_check_request():
        process_request(priority=1)
4. Monitoring and Feedback

Implement real-time monitoring and feedback loops to dynamically adjust thresholds based on system performance:

Feedback Loop:

Use a machine learning model trained on historical data to predict and set load thresholds.
Implement an auto-scaling mechanism that can add or remove resources based on detected load patterns.
PYTHON
def train_model(data):
    # Train a ML model with historical load data
    model = train_machine_learning_model(data)
    return model

def adjust_thresholds(model, current_load):
    # Use the trained model to set new thresholds dynamically
    adjusted_thresholds = get_adjusted_thresholds(model, current_load)
    update_thresholds(adjusted_thresholds)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key is to design an admission control mechanism that can efficiently manage load spikes without impacting critical operations. By implementing rate limiting and dynamic queuing, we ensure that non-critical workloads are shed while essential traffic remains unaffected.

#### Common Mistakes

Failing to separate critical and non-critical traffic effectively.
Not setting appropriate thresholds or monitoring mechanisms.
Overloading the control plane with too many health checks.

For follow-ups:

Discuss how to implement auto-scaling for better resource management.
Explore alternative admission control strategies like burst buffering.

#### Cons
ider edge cases such as intermittent spikes versus sustained high load.

