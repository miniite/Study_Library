---
type: "practice"
difficulty: "Intermediate"
---

# How would you organize a FastAPI project that serves multiple ML models so the code stays maintainable?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Service Architecture and Versioning](<../../../../TOOLS/Python/FastAPI/Service_Architecture_and_Versioning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To organize a FastAPI project serving multiple ML models, we need to ensure that the code is modular, scalable, and maintainable. This involves structuring the project in a way that separates concerns like model loading, data preprocessing, and API endpoints. Let's break down the key steps from first principles.

Firstly, the core idea is to use FastAPI as a framework to create a RESTful API for serving machine learning models. Each ML model will have its own endpoint, allowing us to serve predictions based on different models without mixing their logic or dependencies in one place.

Now, let’s delve into the details:

Project Structure:

Define a clear directory structure that separates concerns such as models, data preprocessing, and API endpoints.

Model Serving Logic:

Each model should be encapsulated within its own module.
Use environment variables or configuration files to specify which model to serve.

Data Preprocessing:

Create utility functions for common preprocessing steps (e.g., scaling, encoding).
Ensure these utilities are reusable across different models and endpoints.

API Endpoints:

Define clear API endpoints for each model.
Use FastAPI’s built-in functionality to handle request validation and responses.

Error Handling:

Implement robust error handling mechanisms.
Log errors and exceptions appropriately.

Testing:

Write unit tests for both the models and the API endpoints.
Include integration testing to ensure that the entire system works as expected.

**Comparison:**

```
Model	Input Schema	Output Schema
model1	age (integer), gender (string)	prediction (float)
model2	text (string)	classification ([positive, negative])
```

**Code:**

```python
from fastapi import FastAPI, HTTPException
import os

app = FastAPI()

# Example model loading function (simplified)
def load_model(model_name):
    if model_name == "model1":
        return Model1()
    elif model_name == "model2":
        return Model2()
    else:
        raise ValueError(f"Unknown model: {model_name}")

@app.get("/predict/{model_name}")
async def predict(model_name: str, input_data: dict):
    try:
        model = load_model(model_name)
        prediction = model.predict(input_data)
        return {"prediction": prediction}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

**Example input / output:**

```text
# Example input and output for model1 predict endpoint
input_data = {"age": 30, "gender": "male"}
response = await predict("model1", input_data)
print(response)  # Output: {"prediction": 0.85}
```

**Data:**

```json
{
  "model1": {
    "input_schema": {
      "age": "integer",
      "gender": "string"
    },
    "output_schema": {
      "prediction": "float"
    }
  },
  "model2": {
    "input_schema": {
      "text": "string"
    },
    "output_schema": {
      "classification": ["positive", "negative"]
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. FastAPI Endpoint
5. Model Selection Based onModel Name
6. Is Model Loaded?
7. Process Input Data withPreprocessing Utilities
8. Call Model's Predict Method
9. Predict Output
10. Return Prediction to User
11. Load Model fromConfiguration

