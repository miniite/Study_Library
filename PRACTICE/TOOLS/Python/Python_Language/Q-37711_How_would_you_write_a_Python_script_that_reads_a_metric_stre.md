---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python script that reads a metric stream and computes an alert only when the value stays above a threshold for a sustained duration?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Flapping Detection](<../../../../TOOLS/Python/Python_Language/Flapping_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the given requirement, we need to write a Python script that monitors incoming metrics in real-time and triggers an alert if the metric's value remains above a specified threshold for a certain period. This involves setting up a mechanism to track when the metric crosses the threshold and then monitor it over time before issuing an alert.

#### Triaging the Question

The question is asking us to implement a system that continuously monitors incoming data points (metrics) and triggers alerts based on conditions defined by a threshold and duration. The core of the problem involves:

Reading real-time metrics.
Storing or keeping track of state information about the metric's value over time.
Determining when the metric remains above a certain threshold for an extended period.

#### Assumptions
The incoming data is in a stream format, likely from some kind of sensor or monitoring service.
We will use time.sleep to simulate real-time data processing and introduce delays between checks.
Alerts are triggered via print statements (for simplicity), but they could be replaced by sending emails, logging messages, etc.

#### First Principles Solution
Initialization: Set up initial variables for the threshold and duration.
Data Stream Simulation: Simulate incoming metric values using a loop or by reading from an external source.
State Tracking: Keep track of whether the current value is above the threshold and if it has been there for long enough to trigger an alert.
Alerting Logic: Trigger an alert when the condition is met.
Code Implementation
PYTHON
import time

# Constants
THRESHOLD = 90
DURATION_SECONDS = 5  # Duration in seconds the metric needs to stay above threshold to trigger an alert
METRIC_VALUE = 85  # Simulated incoming metric value
START_TIME = time.time() - DURATION_SECONDS  # To simulate past values

# Function to check if the condition is met and trigger an alert if so
def check_alert(metric_value, start_time):
    current_time = time.time()
    duration = int(current_time - start_time)

    if metric_value > THRESHOLD:
        if duration >= DURATION_SECONDS:
            print(f"Alert: Metric value {metric_value} has been above threshold for {duration}s.")
    else:
        # Reset the timer when the condition is not met
        global START_TIME
        START_TIME = current_time

# Simulate a stream of metric values
for i in range(100):
    METRIC_VALUE += 5 * (i % 2) - 5 * ((i + 1) % 2)  # Simulating fluctuating value

    if METRIC_VALUE > THRESHOLD:
        check_alert(METRIC_VALUE, START_TIME)

    time.sleep(1)  # Simulate delay between metric readings

#### Closing Summary
and Common Mistakes/Follow-ups
The script continuously checks the incoming metric values against a threshold.
It tracks the duration for which the value has been above the threshold using state variables.
Alerts are triggered only when the condition is met for the specified duration.

#### Common mistakes
include:

Not correctly managing the state of the metric’s value and time.
Failing to reset the start time when the value drops below the threshold.

#### Follow-ups
might involve optimizing for real-time data streams, integrating with a database or external services, or enhancing alert mechanisms (e.g., using an API).

