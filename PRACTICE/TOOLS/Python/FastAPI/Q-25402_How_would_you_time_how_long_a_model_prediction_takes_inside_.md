---
type: "practice"
difficulty: "Basic"
---

# How would you time how long a model prediction takes inside a FastAPI endpoint and return or log that latency?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Real-time and Batch Inference APIs](<../../../../TOOLS/Python/FastAPI/Real-time_and_Batch_Inference_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure the latency of a model prediction inside a FastAPI endpoint, we need to understand the concept of benchmarking in a web application context. The goal is to log or return the time taken by the model to make predictions. This involves capturing the start and end times around the execution of the model's inference logic.

We'll break this down into several steps:

Identify the Model Prediction Code: Locate where the actual prediction happens.
Capture Timing: Measure the time before and after running the prediction code.
Log or Return Latency: Either log the latency to a file or return it in the API response.

Now, let's implement this step by step.

#### Triage

The question asks us to measure the duration of model predictions within a FastAPI endpoint and either log or return that latency. This involves:

Identifying where the prediction happens.
Measuring time using Python’s time module.
Logging or returning the measured time in the response.

We assume the model is already defined and can be called from the FastAPI route.

#### Solution

Here’s a basic implementation:

Import Required Libraries:

PYTHON
import fastapi
import time

Define the Prediction Function: We'll create a simple function that simulates model prediction using the time.sleep method to mimic an actual computation delay.

Create the API Endpoint: Using FastAPI, we’ll create an endpoint where the latency is measured and either logged or returned in the response.

Here’s how you can implement it:

PYTHON
from fastapi import FastAPI, HTTPException

app = FastAPI()

# Simulated model prediction function
def predict_model(input_data):
    # Simulate computation delay (replace with actual model logic)
    time.sleep(2)  # This is just a placeholder for the actual processing time.
    return {"prediction": "some_prediction_value"}

@app.get("/predict")
async def get_prediction(input_data: str):
    start_time = time.time()

    try:
        prediction_result = predict_model(input_data)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

    end_time = time.time()
    latency = end_time - start_time

    # You can log this to a file or any other logging mechanism
    print(f"Prediction Latency: {latency:.4f} seconds")

    return {"prediction": prediction_result, "latency": latency}

#### Closing Summary

In summary, we've created a FastAPI endpoint that measures the latency of model predictions. We captured the start and end times around the model’s prediction logic and either logged or returned this time as part of the response.

#### Common Mistakes/Follow-Ups
Not Handling Exceptions: Ensure you handle any exceptions that might occur during the model prediction.
Logging Mechanism: Decide on a proper logging mechanism instead of print for production code.
Performance Overhead: Be aware that time measurement itself can introduce minor overhead; consider if this needs optimization.

**Code:**

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

def predict_model(input_data):
    # Simulate computation delay (replace with actual model logic)
    time.sleep(2)  # This is just a placeholder for the actual processing time.
    return {"prediction": "some_prediction_value"}

@app.get("/predict")
async def get_prediction(input_data: str):
    start_time = time.time()

    try:
        prediction_result = predict_model(input_data)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

    end_time = time.time()
    latency = end_time - start_time

    print(f"Prediction Latency: {latency:.4f} seconds")

    return {"prediction": prediction_result, "latency": latency}
```

**Example input / output:**

```text
Input: GET /predict?input_data=some_input_value
Output:
{
  "prediction": "some_prediction_value",
  "latency": 2.0456
}
```

