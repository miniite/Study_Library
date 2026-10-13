---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide whether repeated model-inference results in a FastAPI service can be memoized safely, given that model outputs are deterministic but the model version can change under you?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monitoring Model Quality in APIs](<../../../../TOOLS/Python/FastAPI/Monitoring_Model_Quality_in_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine if repeated model-inference results in a FastAPI service can be memoized safely, we need to consider several factors including the determinism of model outputs, the possibility of changing model versions, and the impact on performance and accuracy. The core idea is that memoization can be effective for caching model predictions, but it must account for changes in the model version.

The key questions are:

Are the model's inference results deterministic?
Can the model version change without notice?
What is the expected frequency of input data and how does this affect performance?

Given that your models produce deterministic outputs, memoizing can significantly speed up repeated predictions for the same inputs. However, if there's any possibility that the model version might change, we need to ensure our memoization mechanism accounts for these changes.

**Comparison:**

```
Model Version	Input Data	Cache Result
v1	{"feature1": 0.5, "feature2": 0.3}	Result X
v2	{"feature1": 0.5, "feature2": 0.3}	Result Y (potentially different)
```

**Code:**

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

class MemoizeModelOutput:
    def __init__(self):
        self.cache = {}

    async def predict(self, input_data, model_version):
        if (input_data, model_version) in self.cache:
            return self.cache[(input_data, model_version)]

        # Simulate a prediction call
        result = await simulate_model_prediction(input_data)
        self.cache[(input_data, model_version)] = result
        return result

    async def clear_cache(self):
        self.cache.clear()
```

**Example input / output:**

```text
# Example input and memoized result
input_data = {"feature1": 0.5, "feature2": 0.3}
model_version = "v1"

memoizer = MemoizeModelOutput()

# First prediction
output1 = await memoizer.predict(input_data, model_version)
print("First Prediction:", output1)

# Clear cache and make another request with the same input but a different version
memoizer.clear_cache()
output2 = await memoizer.predict(input_data, "v2")
print("Second Prediction (different version):", output2)

# Note that if the model produces significantly different outputs for v1 and v2, this approach needs to be reconsidered.
```

**Data:**

```json
{
  "input_data": {"feature1": 0.5, "feature2": 0.3},
  "model_version": "v1"
}
```

**Flow:**

1. Yes
2. No
3. Input Request
4. Is Model Version Cached?
5. Return Cache
6. Perform Prediction & CacheResult
7. Response with Cached Output
8. Save Output in Cache

