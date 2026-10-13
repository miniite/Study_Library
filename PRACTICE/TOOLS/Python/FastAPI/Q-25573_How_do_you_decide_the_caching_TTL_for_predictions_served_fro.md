---
type: "practice"
difficulty: "Deep"
---

# How do you decide the caching TTL for predictions served from FastAPI when the model is retrained daily, so you never serve a prediction from a stale model version?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Caching Predictions](<../../../../TOOLS/Python/FastAPI/Caching_Predictions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on the caching Time-to-Live (TTL) for predictions in a FastAPI service where the underlying machine learning model is retrained daily, we must consider several factors to ensure that the system provides accurate and up-to-date predictions while also minimizing the overhead of model serving. The key idea here is to balance freshness with efficiency.

Firstly, let's clarify what the interviewer is probing: they want to understand how you approach a trade-off between serving stale but cached predictions (which can be faster) versus always fetching fresh predictions from the model (which ensures accuracy but may introduce latency). We'll need to make some assumptions about the frequency of retraining and the acceptable staleness duration.

Assuming that the model is retrained daily, we should aim for a TTL that allows us to serve cached results while ensuring they are updated within 24 hours. A common approach is to set the TTL to be slightly shorter than the retraining interval to provide some buffer time for any potential delays or issues in the serving process.

#### First-Principles Solution
Model Retraining Frequency: The model is retrained daily.
Acceptable Staleness Duration: We aim for predictions to be no more than 24 hours old.
Caching Strategy: Use a caching layer (e.g., Redis) to store the results of model predictions.

Let's reason step-by-step:

Determine Model Retraining Time:
If the retraining process takes, say, 6 hours, we need to ensure that new predictions are available within this window.
Set TTL Just Shorter than Retraining Interval:
Set the caching TTL to 23 hours and 59 minutes (1439 minutes). This ensures that cached results are valid for most of the day but allow enough time for retraining if needed.

#### Implementation
in FastAPI

Here's a simplified example of how you might implement this in FastAPI using Redis:

PYTHON
from fastapi import FastAPI, Depends
import redis
import datetime

app = FastAPI()
cache = redis.Redis(host='localhost', port=6379, db=0)

def get_prediction(model_input):
    # Check cache for the prediction
    cached_result = cache.get(str(model_input))

    if cached_result is not None:
        return cached_result.decode('utf-8')

    # If no cache, compute the prediction and store it in Redis with TTL
    result = model.predict(model_input)
    cache.setex(str(model_input), 1439, result)  # 23 hours and 59 minutes TTL

    return result

@app.get('/predict')
def predict(model_input):
    return get_prediction(model_input)

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, setting the caching TTL to just under the model retraining interval ensures that predictions are always as fresh as possible while minimizing latency. A common mistake is setting the TTL too high, which can lead to serving stale predictions.

#### Follow-ups
might include:

How do you handle edge cases where retraining takes longer than expected?
What if your model has different training frequencies for different parts of the day?

