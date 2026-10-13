---
type: "practice"
difficulty: "Deep"
---

# How do you evaluate whether Datadog Watchdog's automatic root-cause suggestion is trustworthy enough to act on during an incident?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Log Parsing and Notebooks](<../../../../TOOLS/Datadog/Datadog_Platform/Log_Parsing_and_Notebooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To assess the trustworthiness of Datadog Watchdog’s automatic root-cause suggestions during a critical incident, we need to consider several factors from both technical and operational perspectives. The key is to evaluate the accuracy, reliability, and consistency of these suggestions in real-time environments.

Firstly, it's important to understand that Datadog Watchdog leverages machine learning models to identify patterns and correlations across various metrics and logs. These models are trained on historical data to predict potential issues before they become incidents. During an incident, the system tries to pinpoint the root cause by correlating events with predefined rules and anomaly detection.

To evaluate trustworthiness, we need to consider these aspects step-by-step:

Data Quality: Ensure that the metrics and logs used for analysis are of high quality and cover the relevant aspects of the application.
Model Performance: Assess how well the machine learning models can predict issues accurately without false positives or negatives.
Rule-Based Logic: Examine the logic behind predefined rules to ensure they align with known issue patterns.
Feedback Loop: Implement a feedback mechanism where users can rate the suggestions, allowing the system to learn and improve over time.

**Comparison:**

```
Metric	Definition	Importance in Evaluation
Precision	Ratio of true positives to all positive predictions	Indicates how reliable the suggestions are when they indicate an incident.
Recall	Ratio of true positives to actual positives	Ensures that incidents are not missed by the system.
```

**Code:**

```python
# Example code snippet for evaluating model performance
from sklearn.metrics import precision_score, recall_score

def evaluate_model_performance(true_labels, predicted_labels):
    precision = precision_score(true_labels, predicted_labels)
    recall = recall_score(true_labels, predicted_labels)
    print(f"Precision: {precision}")
    print(f"Recall: {recall}")

# Mock data for demonstration
true_labels = [0, 1, 0, 1, 0, 1]  # 1 indicates an incident
predicted_labels = [0, 1, 0, 0, 1, 1]

evaluate_model_performance(true_labels, predicted_labels)
```

**Example input / output:**

```text
No specific input-output examples are required for this evaluation, as the process involves evaluating models and feedback mechanisms.
```

**Flow:**

1. Yes
2. No
3. Incident Occurs
4. Is Root Cause Suggested?
5. Review and ValidateSuggestion
6. Act on Suggestion ifConfirmed
7. Check Model PerformanceMetrics
8. Improve Model or Rules Basedon Feedback

