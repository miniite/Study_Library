---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute a simple moving average of the last N CPU readings in Python to smooth out noisy monitoring data?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Percentage Change Calculations](<../../../../TOOLS/Python/Python_Language/Percentage_Change_Calculations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate a simple moving average (SMA) of the last (N) CPU readings, we need to sum up the values and then divide by (N). This method effectively smooths out noisy monitoring data by averaging recent data points.

The core idea is straightforward: take a list of (N) most recent CPU readings, compute their mean, and return that value. The question is probing your understanding of basic Python operations and list handling. We will assume the input is a list of integers or floats representing CPU usage percentages.

First-principles reasoning:

Understand the Input: A list of CPU readings.
Identify (N): Number of recent readings to consider for the SMA.
Compute the Sum: Add up the last (N) elements in the list.
Divide by (N): Average those values to get the SMA.

Let's walk through this step-by-step and write a function that computes the SMA.

**Comparison:**

| Input List | N | SMA Calculation |
|------------|---|-----------------|
| [75, 80, 65, 72, 85, 90, 88, 74, 81, 83] | 3 | (85 + 90 + 88) / 3 = 86.67 |

**Code:**

```python
def simple_moving_average(cpu_readings, N):
    # Ensure we have enough data points
    if len(cpu_readings) < N:
        return None  # or handle differently, e.g., average all available readings

    # Get the last N readings
    recent_readings = cpu_readings[-N:]

    # Calculate the sum of these readings
    total_sum = sum(recent_readings)

    # Compute and return the SMA
    return total_sum / N
```

**Example input / output:**

```text
# Example input and output
print(simple_moving_average(cpu_readings, N))  # Output should be the average of the last 3 readings: (85 + 90 + 88) / 3 = 86.67
```

**Data:**

```python
# Example data
cpu_readings = [75, 80, 65, 72, 85, 90, 88, 74, 81, 83]
N = 3
```

**Flow:**

1. Yes
2. No
3. Input List of CPU Readings
4. Check if len(cpu_readings) >=N
5. Get the last N readings
6. Sum the recent readings
7. Compute SMA = total_sum / N
8. Return None or handledifferently
9. Return SMA

