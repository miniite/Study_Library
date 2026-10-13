---
type: "practice"
difficulty: "Basic"
---

# What is the min-max approach to detecting an outlier, and how would you use Python to flag any value outside a fixed acceptable range for a sensor metric?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Simple Anomaly Detection](<../../../../TOOLS/Python/Python_Language/Simple_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The min-max approach to detecting outliers involves setting a threshold based on the minimum and maximum values observed in a dataset. In this context, we are using Python to flag any value outside an acceptable range for a sensor metric. This method is simple yet effective for identifying anomalies that could indicate issues with the data or underlying processes.

To start, let's define what we're trying to achieve: identify any sensor readings that fall outside a predefined acceptable range. We'll use statistical bounds like the min and max values from historical data as our thresholds. Here are the steps:

Understanding the Problem: Given a series of sensor metrics (e.g., temperature readings), we need to flag any value that is significantly different from the expected range.
Setting Assumptions: We assume that the acceptable range for the sensor metric has already been defined and that this range will not change dynamically during runtime.
Step-by-Step Solution:
Calculate the minimum (min_val) and maximum (max_val) values of the historical data.
Define a function to check if a new incoming value falls within these bounds.

Here's how you can implement this in Python:

PYTHON
# Sample sensor data
sensor_data = [30.5, 31.2, 31.8, 32.1, 34.5, 36.9, 37.2, 38.5]

def find_outliers(sensor_readings):
    # Calculate min and max from the sensor readings
    min_val = min(sensor_readings)
    max_val = max(sensor_readings)

    # Define a function to check if a value is an outlier
    def is_outlier(value):
        return value < min_val or value > max_val

    # Filter out any outliers in the data
    outliers = [value for value in sensor_readings if is_outlier(value)]
    return outliers

# Example usage
outliers = find_outliers(sensor_data)
print(f"Outliers: {outliers}")

This code snippet calculates the min and max values from a list of sensor readings and flags any reading that falls outside this range.

**Comparison:**

```
Sensor Readings	Min Value	Max Value
30.5, 31.2, 31.8, 34.5, 36.9, 37.2, 38.5	30.5	38.5
```

**Code:**

```python
def find_outliers(sensor_readings):
    # Calculate min and max from the sensor readings
    min_val = min(sensor_readings)
    max_val = max(sensor_readings)

    # Define a function to check if a value is an outlier
    def is_outlier(value):
        return value < min_val or value > max_val

    # Filter out any outliers in the data
    outliers = [value for value in sensor_readings if is_outlier(value)]
    return outliers

# Example usage
outliers = find_outliers([30.5, 31.2, 31.8, 34.5, 36.9, 37.2, 38.5])
print(f"Outliers: {outliers}")
```

**Example input / output:**

```plaintext
Outliers: []

This example shows that no outliers were found in the provided dataset.
```

**Data:**

```json
{
  "sensor_readings": [30.5, 31.2, 31.8, 34.5, 36.9, 37.2, 38.5]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Are there sensor readings?
5. Calculate min and max values
6. Check each reading for outliers
7. Flag outliers if any found
8. End

