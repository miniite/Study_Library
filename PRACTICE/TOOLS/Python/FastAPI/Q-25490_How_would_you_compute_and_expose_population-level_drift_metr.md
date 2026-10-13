---
type: "practice"
difficulty: "Deep"
---

# How would you compute and expose population-level drift metrics (like PSI or KL divergence) from features flowing through your FastAPI endpoint without recomputing on every request?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Real-time and Batch Inference APIs](<../../../../TOOLS/Python/FastAPI/Real-time_and_Batch_Inference_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of computing and exposing population-level drift metrics such as Population Stability Index (PSI) or Kullback-Leibler (KL) divergence from features flowing through a FastAPI endpoint without recomputing on every request, we need to implement a solution that efficiently calculates these metrics in an online manner. This involves maintaining state between requests, leveraging caching mechanisms, and possibly employing incremental computation techniques.

The core of the solution lies in setting up a mechanism for collecting data points over time, computing drift metrics periodically or incrementally, and then exposing these metrics via FastAPI endpoints. Let's break down the steps from first principles:

Data Collection: Collect new data samples continuously as they flow through the endpoint.
State Management: Maintain stateful storage to keep track of historical data distributions for each feature.
Incremental Calculation: Use incremental algorithms to update drift metrics without recomputing everything from scratch.
Exposure via API: Expose these metrics through appropriate FastAPI endpoints.

**Comparison:**

```
Feature	Historical Distribution (Example)
feature1	[0.2, 0.3, 0.5]
feature2	[0.4, 0.6, 0.7]
Current State Example
value: 0.6
count: 10
```

**Code:**

```python
from fastapi import FastAPI, HTTPException
import numpy as np
from typing import List

app = FastAPI()

# Example storage for historical data distributions (in real scenarios, use a database)
historical_data_distributions = {}
current_distribution = {}

@app.post("/collect/")
async def collect_data(feature_name: str, new_value: float):
    if feature_name not in current_distribution:
        raise HTTPException(status_code=404, detail="Feature not found")

    # Update the current distribution with new data
    update_current_distribution(current_distribution[feature_name], new_value)

@app.get("/drift/")
async def get_drift(feature_name: str):
    if feature_name not in historical_data_distributions:
        raise HTTPException(status_code=404, detail="No historical data for this feature")

    # Calculate drift using incremental update
    current_distribution = historical_data_distributions[feature_name]
    drift_metric = calculate_drift(current_distribution)

    return {"drift_metric": drift_metric}

def update_current_distribution(distribution: dict, new_value: float):
    if distribution["count"] == 0:
        # Initialize the distribution with a small epsilon to avoid division by zero
        distribution.update({"value": new_value, "count": 1})
    else:
        old_mean = distribution["value"]
        old_count = distribution["count"]

        updated_mean = (old_mean * old_count + new_value) / (old_count + 1)
        updated_count = old_count + 1

        # Update the distribution
        distribution.update({"value": updated_mean, "count": updated_count})

def calculate_drift(distribution: dict):
    # Example implementation for KL divergence calculation
    # In practice, use a library or implement a more sophisticated method
    base_distribution = {"value": 0.5, "count": 100}
    kl_divergence = 0.0

    for bin_idx in range(len(base_distribution["values"])):
        p_i = distribution["values"][bin_idx] / distribution["count"]
        q_i = base_distribution["values"][bin_idx] / base_distribution["count"]

        if p_i > 0 and q_i > 0:
            kl_divergence += (p_i * np.log(p_i / q_i))

    return kl_divergence
```

**Example input / output:**

```text
# Example input and output for the "/collect/" endpoint
@app.post("/collect/")
async def collect_data(feature_name: str, new_value: float):
    if feature_name not in current_distribution:
        raise HTTPException(status_code=404, detail="Feature not found")

    update_current_distribution(current_distribution[feature_name], new_value)

# Example input and output for the "/drift/" endpoint
@app.get("/drift/")
async def get_drift(feature_name: str):
    if feature_name not in historical_data_distributions:
        raise HTTPException(status_code=404, detail="No historical data for this feature")

    current_distribution = historical_data_distributions[feature_name]
    drift_metric = calculate_drift(current_distribution)

    return {"drift_metric": drift_metric}
```

**Data:**

```json
{
  "historical_data_distributions": {
    "feature1": {"values": [0.2, 0.3, 0.5], "count": 300},
    "feature2": {"values": [0.4, 0.6, 0.7], "count": 400}
  },
  "current_distribution": {
    "value": 0.6,
    "count": 10
  }
}
```

**Flow:**

1. Yes
2. No
3. Collect Data
4. Update Current Distribution
5. Store in Historical Distributions
6. Check for Historical Data
7. Calculate Drift
8. Return Drift Metric
9. Initialize or Handle Error

