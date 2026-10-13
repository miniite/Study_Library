---
type: "practice"
difficulty: "Basic"
---

# How would you return a clear error response from FastAPI when the model fails to produce a prediction?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Monitoring Model Quality in APIs](<../../../../TOOLS/Python/FastAPI/Monitoring_Model_Quality_in_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with FastAPI, returning clear and informative error responses is crucial for debugging and maintaining a robust system. In this scenario, if your machine learning (ML) model fails to produce a prediction, we need to ensure that FastAPI provides meaningful information about the issue to the client. Here’s how you can achieve this:

TL;DR

To return a clear error response from FastAPI when the model fails, use exception handling in your API endpoint functions to catch any errors during the prediction process and return an appropriate HTTP status code along with detailed error messages.

#### Triaging the Question

The question is asking about how to handle exceptions within FastAPI endpoints, specifically for scenarios where a machine learning model fails to produce a prediction. The interviewer is probing your understanding of exception handling in Python, FastAPI's response mechanism, and the importance of providing clear feedback to clients.

#### First-Principles Solution
Define Exception Classes: Create custom exception classes to encapsulate different types of errors that might occur during model prediction.
Handle Exceptions in API Endpoints: Use a try-except block within your FastAPI endpoint functions to catch these exceptions and return structured error responses.
Step-by-Step Implementation
Define Custom Exception Class
PYTHON
class ModelPredictionError(Exception):
    def __init__(self, message):
        super().__init__(message)
Endpoint Function with Error Handling
PYTHON
from fastapi import FastAPI, HTTPException
import json

app = FastAPI()

@app.post("/predict")
async def predict(input_data: dict):
    try:
        # Simulate model prediction (replace this with actual ML model logic)
        result = predict_model(input_data)  # Hypothetical function that might fail

        if not result:
            raise ModelPredictionError("Model failed to generate a valid prediction.")

        return {"prediction": result}

    except ModelPredictionError as e:
        error_message = str(e)
        raise HTTPException(status_code=400, detail=json.dumps({"error": error_message}))

#### Closing Summary
and Common Mistakes/Follow-Ups

In this implementation, we defined a custom exception class ModelPredictionError to handle cases where the model fails to produce a valid prediction. We then used a try-except block in our FastAPI endpoint function to catch these exceptions and return a structured error response with an appropriate HTTP status code (400 Bad Request).

#### Common mistakes
include:

Not defining custom exception classes, leading to generic error messages that do not provide enough context.
Ignoring the need for structured error responses, which can lead to inconsistent feedback to clients.

#### Follow-ups
could involve discussing how different exceptions might be handled or integrating logging mechanisms to record errors for later analysis.

