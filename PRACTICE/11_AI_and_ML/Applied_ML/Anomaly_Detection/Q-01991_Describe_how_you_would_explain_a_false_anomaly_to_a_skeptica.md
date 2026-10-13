---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you would explain a false anomaly to a skeptical SRE team and rebuild their trust in the detector.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Multivariate Anomaly Correlation](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Multivariate_Anomaly_Correlation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of explaining false anomalies to a skeptical SRE team, we need to understand that false positives are inherent in any robust anomaly detection system. The goal is not to eliminate them entirely but to ensure they occur infrequently and are easily understood when they do. Here's how I would approach this:

Firstly, it’s important to define what an "anomaly" means in the context of time series data. An anomaly could be a data point that significantly deviates from the expected behavior over a period or pattern. The detection mechanism is based on statistical models and machine learning algorithms that learn normal behaviors and identify deviations.

Now, let's dive into how we can explain a false positive to an SRE team:

Clarify the Detection Process:

Explain that the anomaly detector works by modeling typical behavior through historical data.
Highlight that the model has a threshold for deviation from this norm, which triggers alerts or flags.

Provide Context and Reasoning:

Use clear examples to illustrate how anomalies are detected. For instance, show how a spike in traffic might be flagged as an anomaly if it deviates significantly from the historical pattern.
Discuss the limitations of any statistical model, such as the influence of noise or sudden changes that might not fit into the learned patterns.

Validate and Calibrate:

Explain that the detector is regularly validated using a hold-out dataset to ensure its performance.
Provide evidence of past validation results showing both true positives and false negatives/positives, emphasizing the trade-offs between sensitivity and specificity.

Adjust Thresholds and Parameters:

Suggest adjusting the threshold or parameters based on feedback from the team’s experience.
Offer a step-by-step process to tweak these settings to reduce false positives while maintaining necessary detection accuracy.

Historical Examples:

Present historical cases where false positives occurred, explaining why they were flagged and how they turned out to be normal after further investigation.
Share any lessons learned from these instances to build trust in the system’s reliability over time.

Collaborative Feedback Loop:

Emphasize that the detection process is iterative and that feedback from the team is crucial for continuous improvement.
Propose a mechanism for tracking and reviewing false positives, ensuring they are documented and reviewed regularly.

Technical Justification:

Provide technical insights into why certain data points were flagged as anomalies, such as unusual values or patterns that do not align with historical data.
Use visual aids like time series plots to illustrate the context in which a particular anomaly was detected.

#### Closing Summary
By clearly explaining the detection process and providing transparency on how false positives are handled, we can rebuild trust. Regular validation, adjustments based on feedback, and maintaining a robust documentation system will help ensure that anomalies are accurately identified while minimizing disruptive false positives.

#### Common Mistakes/Follow-Ups
A common mistake is to rely too heavily on automated systems without considering the human aspect of decision-making. Continuous communication and iterative improvement are key to building trust.

**Code:**

```python
# Example Python code for a simple anomaly detection model (Holt-Winters method)
from statsmodels.tsa.holt_winters import ExponentialSmoothing

def detect_anomalies(time_series, alpha=0.3, beta=0.1, gamma=0.1):
    model = ExponentialSmoothing(time_series, seasonal='add', trend='add')
    fitted_model = model.fit(smooth_seasonal=True)
    forecasts = fitted_model.forecast(len(time_series))

    anomalies = []
    for i in range(len(time_series)):
        expected_value = forecasts[i]
        actual_value = time_series[i]
        if abs(actual_value - expected_value) > 3 * np.std(time_series):
            anomalies.append((i, actual_value, expected_value))

    return anomalies
```

**Example input / output:**

```text
| Time | Value |
|------|-------|
| 1    | 100   |
| 2    | 95    |
| ...  | ...   |
| 12   | 110   |
| 13   | 120   |

Anomalies detected:
- Time: 13, Value: 120, Expected Value: 108 (Flagged as an anomaly due to a significant deviation)
```

**Data:**

```json
{
  "time_series": [100, 95, 98, 102, 105, 104, 103, 106, 107, 108, 110, 120]
}
```

