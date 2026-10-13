---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python script that reads CPU readings and prints a warning only after three consecutive high values, to avoid single-spike alerts?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Flapping Detection](<../../../../TOOLS/Python/Python_Language/Flapping_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of sending warnings only for sustained high CPU readings while ignoring single spikes, we can write a Python script that tracks the number of consecutive high readings and triggers a warning once this count reaches three or more. This approach ensures that the system does not generate false alerts due to temporary increases in load.

#### Triaging the Question

The key here is to implement logic that distinguishes between fleeting high CPU usage (spikes) and persistent high usage (warnings). We need to maintain state across multiple readings, count consecutive high values, and only trigger a warning when this count exceeds three. The script should read CPU readings from an input source and process them sequentially.

Step-by-Step Solution
Input Source: Assume the CPU readings are provided as a list of integer values.
State Tracking: Use variables to track consecutive high readings, current high reading status, and previous reading value.
Threshold Definition: Define what constitutes a "high" CPU reading (e.g., above 80%).
Logic Flow:
For each new reading:
Check if it is a high reading.
Increment the consecutive count for high readings.
If three consecutive highs are detected, trigger a warning.

#### Example
Code
PYTHON
def process_cpu_readings(readings):
    warning_count = 0
    previous_reading = None

    for reading in readings:
        if reading > 80:  # High CPU threshold
            warning_count += 1
            current_warning_status = True
        else:
            warning_count = 0
            current_warning_status = False

        if warning_count >= 3 and current_warning_status:
            print("Warning: Sustained high CPU usage detected!")

        previous_reading = reading

# Example usage
cpu_readings = [75, 82, 81, 79, 80, 84, 86, 88, 77]
process_cpu_readings(cpu_readings)

#### Closing Summary
and Common Mistakes/Follow-Ups

The script effectively tracks consecutive high CPU readings and triggers a warning only after three successive readings exceed the threshold. Common mistakes include not resetting the warning_count to zero when a reading drops below the threshold or forgetting to handle the state tracking correctly.

For follow-ups, consider:

Improving Robustness: Handling edge cases like empty lists of readings.
Logging and Debugging: Adding logs for debugging purposes if needed.

#### Performance Considerations
If real-time monitoring is required, ensure the script can efficiently process large datasets or handle high-frequency readings.

