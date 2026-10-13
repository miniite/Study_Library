---
type: "practice"
difficulty: "Intermediate"
---

# How would you return prediction probabilities for all classes from a multiclass classifier in a FastAPI response?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Prediction APIs](<../../../../TOOLS/Python/FastAPI/Time-series_Prediction_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To return prediction probabilities for all classes from a multiclass classifier in a FastAPI response, we need to ensure that our application is both efficient and robust. The goal is to set up a FastAPI endpoint that receives input data, processes it through the model, and returns the class probabilities in a well-structured JSON format.

#### Triage

The question essentially asks for a way to implement an API endpoint using FastAPI to return prediction probabilities from a multiclass classifier. This involves setting up a FastAPI application, defining a route to handle incoming requests, processing those requests through a model (which could be any trained machine learning model), and then returning the predicted class probabilities.

#### Assumptions
The multiclass classifier is already trained.
The input data format is known and consistent.
The FastAPI application is running locally on an accessible server.
Step-by-Step Solution
1. Setting Up the FastAPI Application

First, we need to install FastAPI and create a simple application structure:

BASH
pip install fastapi uvicorn

Create a Python file (e.g., main.py) with the following content:

PYTHON
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class InputData(BaseModel):
    input_data: list  # Assuming input data is in JSON format, e.g., [1.0, -2.3]

@app.post("/predict_probabilities")
async def predict_probabilities(input_data: InputData):
    # Load the pre-trained model
    from sklearn.externals import joblib
    model = joblib.load("multiclass_model.pkl")

    # Extract input data
    inputs = [input_data.input_data]  # Ensure it's a list of lists

    # Make prediction and get probabilities
    probabilities = model.predict_proba(inputs)

    return {"probabilities": probabilities.tolist()}
2. Running the FastAPI Application

To run the application, use:

BASH
uvicorn main:app --reload

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we have set up a FastAPI application with an endpoint that processes input data and returns class probabilities using a pre-trained model. Key points include defining the input schema, loading the model from disk, making predictions, and returning the results in a structured JSON format.

#### Common Mistakes
Incorrect Input Data Handling: Ensure that the input data is correctly formatted as expected by your model.
Model Loading Issues: Make sure the path to the saved model file (multiclass_model.pkl) is correct.
Dependency Management: Ensure all required packages (FastAPI, Pydantic) are installed.

If asked for further details or optimizations:

Discuss how you would handle large datasets efficiently.

#### Cons
ider integrating a database for storing input and output data.
Explore deploying the application on a more robust server like AWS or Heroku.

