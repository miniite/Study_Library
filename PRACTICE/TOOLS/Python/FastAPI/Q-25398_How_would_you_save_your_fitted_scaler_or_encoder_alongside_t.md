---
type: "practice"
difficulty: "Intermediate"
---

# How would you save your fitted scaler or encoder alongside the model and reload both in FastAPI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Model Consistency Across Deploys](<../../../../TOOLS/Python/FastAPI/Model_Consistency_Across_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To save a fitted scaler or encoder alongside an ML model in a FastAPI application, we need to ensure that both components can be saved and loaded consistently. This is crucial because scalers and encoders are often used to preprocess data before feeding it into the model. Here’s how you can achieve this:

Firstly, let's break down what the question is asking:

Save a fitted Scaler or Encoder: We need to save an instance of a scaler (e.g., StandardScaler) or encoder (e.g., OneHotEncoder) that has been trained on some data.
Save the Model: We also need to save the ML model itself, which is likely trained using the preprocessed data from step 1.
Load Both Components in FastAPI: When starting the application, we should load both the scaler or encoder and the model so that predictions can be made on new data.

#### Assumptions

We are using Python with scikit-learn for the ML model and scalers/encoders.
The model is being used to make predictions based on some input features.
We have a FastAPI application where we want to serve the model as an API endpoint.

Now, let's walk through the solution step-by-step:

Step 1: Import Necessary Libraries
PYTHON
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.pipeline import Pipeline
from fastapi import FastAPI, HTTPException
import joblib
Step 2: Save the Scaler/Encoder and Model

Assume we have a trained model model and a fitted scaler or encoder scaler.

PYTHON
# Save the scaler
joblib.dump(scaler, 'scaler.pkl')

# Save the model
joblib.dump(model, 'model.pkl')
Step 3: Load Scaler/Encoder and Model in FastAPI

In your FastAPI application, you need to load these components during startup.

PYTHON
app = FastAPI()

@app.on_event("startup")
async def load_model_and_scaler():
    global model, scaler

    try:
        # Load the scaler
        scaler = joblib.load('scaler.pkl')

        # Load the model
        model = joblib.load('model.pkl')

    except FileNotFoundError as e:
        raise HTTPException(status_code=500, detail="Model or Scaler not found")
Step 4: Create a Prediction Endpoint

Now that both components are loaded, you can create an endpoint to receive input data and make predictions.

PYTHON
@app.post("/predict/")
async def predict(data: Dict[str, Any]):
    try:
        # Preprocess the input data using the scaler
        preprocessed_data = scaler.transform([data])

        # Make prediction
        prediction = model.predict(preprocessed_data)

        return {"prediction": prediction.tolist()}

    except Exception as e:
        raise HTTPException(status_code=500, detail="Error during prediction")

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we ensure that both the scaler and the model are saved and loaded properly. The key is using joblib to serialize/deserialize the objects.

#### Common mistakes
include:

Forgetting to save or load the components.
Not handling exceptions correctly, which could lead to application crashes.
Using incorrect file names when saving/loading.

For follow-ups, you might want to consider:

Versioning: How do we handle different versions of models and scalers?
Security: How can we ensure that only authorized users can load the model/scaler files?

