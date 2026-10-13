---
type: "practice"
difficulty: "Basic"
---

# What is the difference between accuracy and precision when talking about data quality?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Quality Fundamentals](<../../../../02_Data_Engineering/Data_Quality/Validation/Data_Quality_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of data quality, understanding the distinction between accuracy and precision is crucial. Accuracy refers to how close a value is to its true or actual value, while precision is about the level of detail or exactness in the representation of that value. This summary sets up the core concepts we will explore.

To elaborate, accuracy means that your data values are correct and error-free; they align with the real-world truth. For example, if a temperature sensor reads 20°C when the actual temperature is indeed 20°C, it is accurate. On the other hand, precision concerns how consistent or repeatable these readings are, regardless of their accuracy.

Imagine you're measuring the weight of an object multiple times. If your measurements consistently read exactly 150 grams each time, but they are off by a few grams from the actual weight, then your data is precise but not accurate. Conversely, if your measurements vary widely, sometimes reading 148 grams and other times 152 grams, it shows low precision.

Let's consider an example to illustrate further:

Accuracy Example: A weight scale that consistently reads a person’s weight as exactly their actual weight (e.g., 70 kg when the true weight is 70 kg).
Precision Example: A temperature sensor that fluctuates between 25°C and 26°C but never deviates from these values, even if the actual temperature is different.

In practice, achieving high accuracy often requires precision as well. However, in environments where precision is paramount (e.g., financial transactions), it’s possible to focus on consistency without necessarily ensuring absolute correctness.

**Comparison:**

```
Measurement	True Value (°C)	Deviation from True
24.9	25	-0.1
25.0	25	0
25.1	25	0.1
26.0	25	1
27.0	25	2
23.8	25	-1.2
25.0	25	0
25.0	25	0
```

**Code:**

```python
# Example of a simple function to check data quality metrics

def validate_data_quality(data):
    # Assume `data` is a list of temperature readings in Celsius
    true_value = 25.0  # True value for comparison

    accurate_readings = [1 if abs(x - true_value) < 0.1 else 0 for x in data]
    precision_readings = len(set(data)) <= 2  # Check for high precision (low variability)

    accuracy = sum(accurate_readings) / len(data)
    precision = "High" if precision_readings else "Low"

    return {"accuracy": accuracy, "precision": precision}
```

**Example input / output:**

```text
data = [24.9, 25.0, 25.1, 26.0, 27.0, 23.8, 25.0, 25.0]

result = validate_data_quality(data)
print(result)  # Output: {"accuracy": 0.4, "precision": "Low"}
```

**Data:**

```json
[
    24.9,
    25.0,
    25.1,
    26.0,
    27.0,
    23.8,
    25.0,
    25.0
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Collection
6. Is Measurement Accurate?
7. High Accuracy
8. Is Measurement Precise?
9. High Precision
10. Low Precision

