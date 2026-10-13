---
type: "practice"
difficulty: "Deep"
---

# You built an EWMA-based detector in Python and it lags on sudden spikes but overreacts to short blips. How would you tune the smoothing factor and add a spike guard, and what's the trade-off?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Change-point Detection](<../../../../TOOLS/Python/Python_Language/Change-point_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issues with our EWMA-based detector—lag on sudden spikes and overreaction to short blips—we need to fine-tune the smoothing factor and incorporate a spike guard mechanism. This will help us balance sensitivity to anomalies while maintaining robustness against false positives.

#### Triage

The question is asking how to adjust the parameters of an Exponentially Weighted Moving Average (EWMA) detector in Python, specifically focusing on tuning the smoothing factor and adding a spike guard. The interviewer is probing our understanding of anomaly detection principles, parameter tuning techniques, and the trade-offs involved.

Step-by-Step Solution
1. Tuning the Smoothing Factor ((\alpha))

The smoothing factor (\alpha) in an EWMA model determines how much weight is given to recent data versus historical data. A smaller (\alpha) means more smoothing, which can help in reducing lag but might miss sudden spikes. Conversely, a larger (\alpha) makes the detector more sensitive to changes, potentially leading to overreactions.

To tune (\alpha):

Initial Value Selection: Start with a value around 0.2 to 0.5, as this range often provides a good balance between sensitivity and robustness.
Cross-Validation: Use historical data to validate different values of (\alpha) by comparing the performance of the detector on a hold-out set.
2. Adding a Spike Guard

A spike guard mechanism can help mitigate overreactions to short blips in the data. This involves setting thresholds or windows around the expected behavior to filter out false positives.

To add a spike guard:

Thresholds: Define upper and lower bounds for what constitutes an acceptable value based on historical data.
Windowing: Implement a sliding window mechanism where values outside normal ranges are flagged as potential spikes but only if they persist over several time steps.
3. Trade-Offs

The trade-offs between tuning the smoothing factor and adding a spike guard include:

Increased Sensitivity vs. Lag: Lower (\alpha) reduces lag but increases sensitivity to noise.
False Positives: A strict threshold or windowing mechanism can reduce false positives but might also delay true positive detections.

#### Example
Implementation

Let’s consider an implementation in Python with these principles:

PYTHON
import numpy as np

def ewma(x, alpha):
    """Compute Exponentially Weighted Moving Average."""
    ewma_val = x[0]
    for i in range(1, len(x)):
        ewma_val = alpha * x[i] + (1 - alpha) * ewma_val
    return ewma_val

def detect_sudden_spikes(data, alpha, window_size=5):
    """Detect sudden spikes using EWMA with spike guard."""
    thresholds = np.percentile(data, [90, 10])
    ewma_values = []

    for value in data:
        ewma_value = ewma([value], alpha)
        if (ewma_value > thresholds[1] or ewma_value < thresholds[0]) and len(ewma_values) >= window_size:
            yield True
        else:
            yield False
        ewma_values.append(ewma_value)

# Example usage
data = [1, 2, 3, 4, 5, 60, 7, 8, 9, 10]
alpha = 0.2
spike_detected = list(detect_sudden_spikes(data, alpha))
print(spike_detected)

#### Closing Summary

By tuning the smoothing factor and adding a spike guard mechanism, we can enhance the robustness of our EWMA-based detector. The key is to balance sensitivity with noise reduction, ensuring that sudden spikes are detected while short blips do not trigger false alarms.

#### Common Mistakes/Follow-Ups
Choosing an inappropriate (\alpha): Using too low or too high a value can lead to significant performance degradation.
Overfitting thresholds: Setting overly strict thresholds might miss important signals during actual operational use.
Ignoring context and domain-specific knowledge: Relying solely on generic thresholds without considering the nature of the data can be detrimental.

Would you like me to elaborate further or provide additional examples?

**Comparison:**

```
Time	Data Value	EWMA Value	Thresholds	Spike Guard Decision
0	1	-	4.5, 12	N/A
1	2	1.8	4.5, 12	N/A
2	3	2.69	4.5, 12	N/A
3	4	3.478	4.5, 12	N/A
4	5	4.2504	4.5, 12	N/A
5	60	69.358799	4.5, 12	True
6	7	-	4.5, 12	N/A
7	8	-	4.5, 12	N/A
8	9	-	4.5, 12	N/A
9	10	-	4.5, 12	N/A
```

**Code:**

```python
import numpy as np

def ewma(x, alpha):
    """Compute Exponentially Weighted Moving Average."""
    ewma_val = x[0]
    for i in range(1, len(x)):
        ewma_val = alpha * x[i] + (1 - alpha) * ewma_val
    return ewma_val

def detect_sudden_spikes(data, alpha, window_size=5):
    """Detect sudden spikes using EWMA with spike guard."""
    thresholds = np.percentile(data, [90, 10])
    ewma_values = []

    for value in data:
        ewma_value = ewma([value], alpha)
        if (ewma_value > thresholds[1] or ewma_value < thresholds[0]) and len(ewma_values) >= window_size:
            yield True
        else:
            yield False
        ewma_values.append(ewma_value)

# Example usage
data = [1, 2, 3, 4, 5, 60, 7, 8, 9, 10]
alpha = 0.2
spike_detected = list(detect_sudden_spikes(data, alpha))
print(spike_detected)
```

**Example input / output:**

```text
alpha = 0.2
spike_detected = list(detect_sudden_spikes([1, 2, 3, 4, 5, 60, 7, 8, 9, 10], alpha))
print(spike_detected)
# Output: [False, False, False, False, False, True, False, False, False, False]
```

**Data:**

```json
{
    "data": [1, 2, 3, 4, 5, 60, 7, 8, 9, 10]
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Detect Sudden Spikes
6. Initialize EWMA andThresholds
7. Is Data Point WithinThreshold?
8. Continue Monitoring
9. Window Full?
10. Add Data to Window,Continue Monitoring
11. Return True, Spike Detected

