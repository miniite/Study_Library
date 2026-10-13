---
type: "practice"
difficulty: "Intermediate"
---

# How would you add a simple endpoint in FastAPI to reload the model without restarting the whole service?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Prediction APIs](<../../../../TOOLS/Python/FastAPI/Time-series_Prediction_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create an endpoint in FastAPI that allows reloading a trained machine learning model without restarting the entire service, we need to follow several steps from first principles. This involves understanding how FastAPI handles asynchronous operations and how to dynamically reload models using Python's joblib or similar libraries.

#### Triaging the Question

The question asks about adding an endpoint in FastAPI that can trigger a model reload. The interviewer is probing your knowledge of FastAPI's architecture, asynchronous programming concepts, and dynamic reloading mechanisms. We will make some assumptions: we are using a trained machine learning model saved as model.pkl, and this model needs to be reloaded whenever new data arrives.

Step-by-Step Solution
Set Up the Basic FastAPI Application: First, ensure you have FastAPI set up with Uvicorn or Gunicorn for running your application.
Create a Model Class: Define a class that will handle loading and unloading the model.
Use ASGI State for Dynamic Reloads: FastAPI runs on an ASGI server like Uvicorn, which supports stateful operations across requests. We can use this to keep track of whether the model is currently loaded or not.
Create the Endpoint: Define a new endpoint that will send a request to reload the model.
Code Implementation
PYTHON
from fastapi import FastAPI, Request
import joblib
from pathlib import Path
import asyncio

app = FastAPI()

# Global state for model loading/unloading
model_path = "path/to/model.pkl"
model = None  # Initialize as None

class ModelManager:
    def __init__(self):
        self.model = None

    async def load_model(self) -> object:
        global model
        if not model:  # Only load the model once
            model = joblib.load(model_path)
        return model

    async def unload_model(self):
        global model
        model = None  # Unload the model

model_manager = ModelManager()

@app.get("/predict")
async def predict(request: Request):
    """Endpoint to make predictions using the loaded model."""
    global model
    if not model:
        raise Exception("Model is not loaded. Please use /reload endpoint.")

    input_data = request.json()  # Assuming the data comes in as JSON
    prediction = model.predict([input_data])[0]
    return {"prediction": prediction}

@app.get("/reload")
async def reload_model(request: Request):
    """Endpoint to trigger model reloading."""
    global model
    await model_manager.unload_model()
    await asyncio.sleep(1)  # Simulate delay for model reloading
    await model_manager.load_model()

    return {"status": "Model reloaded successfully"}

@app.on_event("startup")
async def startup_event():
    """Load the model when the application starts."""
    global model
    if not model:
        await model_manager.load_model()

@app.on_event("shutdown")
async def shutdown_event():
    """Unload the model when the application stops."""
    global model
    if model:
        await model_manager.unload_model()

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we have created a FastAPI application with two endpoints: /predict for making predictions using the model and /reload to dynamically reload the model without restarting the service. The key idea here is leveraging ASGI's stateful capabilities to manage the lifecycle of the model.

#### Common Mistakes

Failing to Use Asynchronous Functions: Ensure all operations, especially IO-bound tasks like reloading models, are asynchronous.
Overwriting Model Globally Without Locking: Be cautious about global variables and ensure thread safety if running in multi-threaded environments.
Not Handling Errors Gracefully: Always handle exceptions properly, especially during model loading or unloading.
Additional Considerations
Testing the Endpoints: Make sure to test both /predict and /reload endpoints thoroughly using tools like curl or Postman.
Scalability: For production use, consider implementing a health check endpoint and logging mechanisms for better monitoring.

**Code:**

```python
from fastapi import FastAPI, Request
import joblib
from pathlib import Path
import asyncio

app = FastAPI()

# Global state for model loading/unloading
model_path = "path/to/model.pkl"
model = None  # Initialize as None

class ModelManager:
    def __init__(self):
        self.model = None

    async def load_model(self) -> object:
        global model
        if not model:  # Only load the model once
            model = joblib.load(model_path)
        return model

    async def unload_model(self):
        global model
        model = None  # Unload the model

model_manager = ModelManager()

@app.get("/predict")
async def predict(request: Request):
    """Endpoint to make predictions using the loaded model."""
    global model
    if not model:
        raise Exception("Model is not loaded. Please use /reload endpoint.")

    input_data = request.json()  # Assuming the data comes in as JSON
    prediction = model.predict([input_data])[0]
    return {"prediction": prediction}

@app.get("/reload")
async def reload_model(request: Request):
    """Endpoint to trigger model reloading."""
    global model
    await model_manager.unload_model()
    await asyncio.sleep(1)  # Simulate delay for model reloading
    await model_manager.load_model()

    return {"status": "Model reloaded successfully"}

@app.on_event("startup")
async def startup_event():
    """Load the model when the application starts."""
    global model
    if not model:
        await model_manager.load_model()

@app.on_event("shutdown")
async def shutdown_event():
    """Unload the model when the application stops."""
    global model
    if model:
        await model_manager.unload_model()
```

