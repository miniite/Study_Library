---
type: "practice"
difficulty: "Intermediate"
---

# How would you support both a single prediction and a batch prediction on the same FastAPI service using two endpoints?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Prediction APIs](<../../../../TOOLS/Python/FastAPI/Time-series_Prediction_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To support both a single prediction and a batch prediction in a FastAPI service, we need to design our API such that it can handle different input formats efficiently. This involves setting up two distinct endpoints, one for handling single predictions and another for batch predictions. Let's break down the solution step by step.

First, we'll clarify what the interviewer is probing:

Endpoint Design: The ability to define multiple endpoints with different input/output structures.
Input Handling: Efficiently parsing JSON inputs into appropriate data formats (e.g., list of dictionaries vs. dictionary).

#### Performance Considerations
Ensuring that both single and batch predictions are handled effectively without unnecessary overhead.

We'll assume the following:

The model expects the same input format for both types of requests.
Single prediction input is a JSON object, while batch prediction input is an array of JSON objects.

Now let's reason through this from first principles:

Step 1: Define the Model and Input Structures

For simplicity, let’s assume we have a basic model that predicts house prices based on features like size and location. The input for both single and batch predictions would be structured as follows:

JSON
{
    "size": float,
    "location": str
}
Step 2: Set Up the FastAPI Service

We'll create two endpoints using FastAPI. One endpoint will handle a single prediction, while another will handle multiple predictions.

Here’s how we can implement this:

Single Prediction Endpoint

URL: /predict
Method: POST
Input: JSON object with features for a single instance
Output: Predicted price

Batch Prediction Endpoint

URL: /batch-predict
Method: POST
Input: Array of JSON objects, each representing an instance's features
Output: List of predicted prices
Step 3: Implement the Endpoints in FastAPI

Below is a sample implementation:

PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import numpy as np  # Example: Using NumPy for simplicity
import joblib  # Assuming we load our model using this

# Load the model
model = joblib.load('path/to/model.pkl')

app = FastAPI()

class PredictionInput(BaseModel):
    size: float
    location: str

@app.post("/predict")
async def predict_single(input_data: PredictionInput):
    """
    Predicts the price for a single instance.
    """
    input_dict = input_data.dict()
    prediction = model.predict([input_dict])
    return {"prediction": prediction[0]}

@app.post("/batch-predict")
async def batch_predict(inputs: list[PredictionInput]):
    """
    Predicts prices for multiple instances in a batch.
    """
    inputs_dicts = [input_.dict() for input_ in inputs]
    predictions = model.predict([d['size'] for d in inputs_dicts])
    return {"predictions": predictions.tolist()}
Step 4: Test the Endpoints

Let’s test both endpoints with some sample data:

Single Prediction:
JSON
{
    "size": 1500.0,
    "location": "Urban"
}

Expected Output:

JSON
{
    "prediction": [250000.0]
}
Batch Prediction:
JSON
[
    {"size": 1500.0, "location": "Urban"},
    {"size": 2000.0, "location": "Suburban"}
]

Expected Output:

JSON
{
    "predictions": [250000.0, 300000.0]
}

#### Closing Summary
and Common Mistakes/Follow-ups

In this solution, we’ve effectively created a FastAPI service with two endpoints to handle both single and batch predictions. The key is to ensure that the input handling logic is clear and efficient.

#### Common Mistakes

Failing to properly define Pydantic models for input validation.
Mixing up request types (e.g., POST vs. GET) in endpoint design.
Overcomplicating model loading or prediction logic without proper error handling.

For follow-ups, the interviewer might ask about:

How to handle errors and edge cases.
Performance optimizations, especially if dealing with large batches.
Security considerations when deploying this service externally.

**Comparison:**

```
Endpoint	URL	Method	Input Type	Output Type
Single	/predict	POST	PredictionInput	Float
Batch	/batch-predict	POST	List[PredictionInput]	List[Float]
```

**Code:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import joblib

# Load the model
model = joblib.load('path/to/model.pkl')

app = FastAPI()

class PredictionInput(BaseModel):
    size: float
    location: str

@app.post("/predict")
async def predict_single(input_data: PredictionInput):
    input_dict = input_data.dict()
    prediction = model.predict([input_dict])
    return {"prediction": prediction[0]}

@app.post("/batch-predict")
async def batch_predict(inputs: list[PredictionInput]):
    inputs_dicts = [input_.dict() for input_ in inputs]
    predictions = model.predict([d['size'] for d in inputs_dicts])
    return {"predictions": predictions.tolist()}
```

**Example input / output:**

```text
# Single Prediction Input & Output
Single input:
{
    "size": 1500.0,
    "location": "Urban"
}

Expected output:
{
    "prediction": [250000.0]
}

# Batch Prediction Input & Output
Batch inputs:
[
    {
        "size": 1500.0,
        "location": "Urban"
    },
    {
        "size": 2000.0,
        "location": "Suburban"
    }
]

Expected output:
{
    "predictions": [250000.0, 300000.0]
}
```

**Data:**

```json
{
    "single_prediction_input": {
        "size": 1500.0,
        "location": "Urban"
    },
    "batch_prediction_input": [
        {
            "size": 1500.0,
            "location": "Urban"
        },
        {
            "size": 2000.0,
            "location": "Suburban"
        }
    ]
}
```

**Flow:**

1. User Request
2. Is Single or Batch?
3. Single Predict
4. Batch Predict
5. Extract Input, Make Prediction,Return Result
6. List of Inputs, Extract Each,Make Predictions, AggregateResults, Return List

