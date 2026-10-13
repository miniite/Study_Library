---
type: "practice"
difficulty: "Deep"
---

# Your CI runs the anomaly detector against a fixed labeled dataset, but prod telemetry drifted and the dataset is stale; how would you keep the evaluation dataset representative over time?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Infrastructure Drift Detection](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Infrastructure_Drift_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your CI/CD pipeline's anomaly detection model remains effective over time, it’s crucial to maintain a representative evaluation dataset that reflects current production telemetry trends. The challenge here is that the fixed labeled dataset used to evaluate the model becomes stale due to prod telemetry drift. To address this, we can implement a continuous learning approach where new data is periodically incorporated into the training and evaluation datasets.

This involves several steps: monitoring telemetry changes, periodically retraining models, updating the evaluation dataset, and ensuring robust testing strategies.

**Comparison:**

```
Existing Eval Dataset	Updated Eval Dataset
Metric Value	New Data
50	75
60	80
```

**Code:**

```python
# Example of a Python script to update the evaluation dataset

def update_evaluation_dataset(current_prod_telemetry, existing_eval_dataset):
    # Filter out anomalies or significant drifts in telemetry data
    filtered_prod_telemery = filter_drifted_data(current_prod_telemetry)

    # Combine new telemetry with existing eval dataset
    updated_eval_dataset = merge_datasets(filtered_prod_telemery, existing_eval_dataset)

    return updated_eval_dataset

def filter_drifted_data(data):
    # Implement logic to identify and exclude drifted data points
    pass

def merge_datasets(new_data, old_data):
    # Merge new telemetry with the existing evaluation dataset
    merged_data = old_data + new_data
    return merged_data
```

**Example input / output:**

```text
Updated Evaluation Dataset:
- timestamp: "2023-10-01"
  metric_value: 50
  label: "normal"
- timestamp: "2023-10-02"
  metric_value: 60
  label: "anomaly"
- timestamp: "2023-10-03"
  metric_value: 75
  label: "anomaly"
- timestamp: "2023-10-04"
  metric_value: 80
  label: "normal"
```

**Data:**

```yaml
existing_eval_dataset:
  - timestamp: "2023-10-01"
    metric_value: 50
    label: "normal"
  - timestamp: "2023-10-02"
    metric_value: 60
    label: "anomaly"

current_prod_telemetry:
  - timestamp: "2023-10-03"
    metric_value: 75
    label: "anomaly"
  - timestamp: "2023-10-04"
    metric_value: 80
    label: "normal"
```

**Flow:**

1. Production Telemetry
2. Filter Drifted Data
3. Update Evaluation Dataset
4. Retrain Model
5. Evaluate New Model
6. Cycle Continues|PeriodicallyUpdate Datasets

