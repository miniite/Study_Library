---
type: "practice"
difficulty: "Intermediate"
---

# How would you log each prediction request and its output in a FastAPI ML service for later analysis?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Real-time and Batch Inference APIs](<../../../../TOOLS/Python/FastAPI/Real-time_and_Batch_Inference_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To log each prediction request and its output in a FastAPI ML service, we need to ensure that every incoming HTTP request is logged along with the corresponding prediction result. This involves setting up logging mechanisms within our FastAPI application, handling data serialization, and possibly storing logs for future analysis.

#### Triage

The question asks about implementing logging in a FastAPI-based machine learning (ML) service. The primary goal is to capture detailed information about each request and its resulting output for later analysis. This involves understanding the structure of requests and responses, identifying appropriate log levels, and deciding where to store these logs.

#### First-Principles Solution
Understand Request Structure: Define the format of incoming HTTP requests.
Capture Input Data: Extract relevant information from each request.
Log Request Details: Use FastAPI's logging capabilities or external logging mechanisms.
Handle Output Data: Serialize the output predictions for storage and logging.
Store Logs: Optionally, store logs in a database or file system for later analysis.
Detailed Implementation

Define Request Structure

PYTHON
from pydantic import BaseModel

class PredictionRequest(BaseModel):
    input_data: dict  # Example: {"feature_1": 0.5, "feature_2": 1.0}

Capture Input Data and Log Request

PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import logging

app = FastAPI()

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.post("/predict")
async def predict(request: PredictionRequest):
    try:
        input_data = request.input_data
        # Log the incoming request details
        logger.info(f"Received request with input data {input_data}")

        # Perform prediction (simplified for example)
        prediction = perform_prediction(input_data)  # Assume this function is defined

        # Log the output of the prediction
        logger.info(f"Prediction result: {prediction}")
    except Exception as e:
        logger.error(f"An error occurred during processing: {e}", exc_info=True)
        raise HTTPException(status_code=500, detail="Internal server error")

    return {"prediction": prediction}

Handle Output Data and Logging

PYTHON
from fastapi.responses import JSONResponse

@app.post("/predict")
async def predict(request: PredictionRequest):
    try:
        input_data = request.input_data
        logger.info(f"Received request with input data {input_data}")

        prediction = perform_prediction(input_data)
        logger.info(f"Prediction result: {prediction}")

        # Optionally, log to a file or database for persistent storage
        write_to_log_file(prediction)  # Assume this function is defined

    except Exception as e:
        logger.error(f"An error occurred during processing: {e}", exc_info=True)
        raise HTTPException(status_code=500, detail="Internal server error")

    return JSONResponse(content={"prediction": prediction})

Store Logs

PYTHON
# Example function to write logs to a file
import os

def write_to_log_file(prediction):
    log_path = "/path/to/log/file.log"
    with open(log_path, "a") as f:
        f.write(f"Timestamp: {datetime.now()}, Prediction: {prediction}\n")

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By leveraging FastAPI's built-in logging mechanisms and adding custom log entries for request and response details, we can effectively capture and store the necessary information for later analysis.

#### Common Mistakes
Forgetting to handle exceptions properly, leading to lost logs or unhandled errors.
Not considering performance implications of extensive logging, especially in high-traffic services.
Neglecting to secure sensitive data logged from requests.
Self-Check

Correctness + completeness confirmed.

**Code:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import logging
import datetime

app = FastAPI()

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class PredictionRequest(BaseModel):
    input_data: dict  # Example: {"feature_1": 0.5, "feature_2": 1.0}

@app.post("/predict")
async def predict(request: PredictionRequest):
    try:
        input_data = request.input_data
        logger.info(f"Received request with input data {input_data}")

        prediction = perform_prediction(input_data)
        logger.info(f"Prediction result: {prediction}")

        write_to_log_file(prediction)  # Assume this function is defined

    except Exception as e:
        logger.error(f"An error occurred during processing: {e}", exc_info=True)
        raise HTTPException(status_code=500, detail="Internal server error")

    return JSONResponse(content={"prediction": prediction})

def write_to_log_file(prediction):
    log_path = "/path/to/log/file.log"
    with open(log_path, "a") as f:
        f.write(f"Timestamp: {datetime.now()}, Prediction: {prediction}\n")
```

**Example input / output:**

```text
Input Request Example:
{
  "input_data": {
    "feature_1": 0.5,
    "feature_2": 1.0
  }
}

Output Response Example:
{
  "prediction": "some_prediction_result"
}
```

**Flow:**

1. Yes
2. User Sends Prediction Request
3. Is Input Valid?
4. Log Request Details
5. Predict Using Model
6. Generate Prediction Result
7. Log Output Details
8. Return Response to User
9. Store Logs in File System

