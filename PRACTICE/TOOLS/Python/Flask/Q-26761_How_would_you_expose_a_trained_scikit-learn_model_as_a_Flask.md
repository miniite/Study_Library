---
type: "practice"
difficulty: "Basic"
---

# How would you expose a trained scikit-learn model as a Flask /predict endpoint that accepts JSON features and returns a prediction?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Loading Models Efficiently](<../../../../TOOLS/Python/Flask/Loading_Models_Efficiently.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To expose a trained scikit-learn model as a Flask /predict endpoint, we need to create a simple web server using Flask that can accept JSON-formatted feature data, process it through our trained model, and return the prediction in JSON format. This task involves setting up a basic Flask application, integrating the model, and defining an API endpoint.

First, let's break down the problem into smaller steps:

Install necessary libraries.
Train your scikit-learn model (assuming you have already done this).
Set up a Flask application.
Define a route for the prediction endpoint.
Process incoming JSON data and return the prediction.

Now, we will proceed with each step in detail:

Step 1: Install Necessary Libraries

Ensure that scikit-learn, Flask, and json are installed. You can install them using pip if you haven't already:

BASH
pip install scikit-learn flask
Step 2: Train Your Model (Assumed)

For the sake of this example, let's assume we have a simple scikit-learn model that is trained on some data.

Step 3: Set Up Flask Application

Create a Python file and set up the basic structure of your Flask application. We'll use Flask to handle HTTP requests.

PYTHON
from flask import Flask, request, jsonify
import joblib

app = Flask(__name__)

# Load the trained model (assumed you have saved it as 'model.pkl')
model_path = 'path/to/model.pkl'
model = joblib.load(model_path)

@app.route('/predict', methods=['POST'])
def predict():
    # Get JSON data from request
    json_data = request.get_json()

    # Extract features and prepare them for the model (assuming `data` is the key)
    input_features = [json_data['data']]

    # Make prediction using the loaded model
    prediction = model.predict(input_features)

    return jsonify({'prediction': prediction.tolist()})
Step 4: Define a Route for Prediction Endpoint

The route /predict will accept POST requests and expects JSON data containing feature values. The response will be in JSON format, containing the predicted label.

Step 5: Process Incoming JSON Data and Return the Prediction

In this example, we assume that the incoming JSON has a key data which contains the input features as a list or array. We extract these features, pass them to the model for prediction, and return the result in JSON format.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, we created a simple Flask application with one endpoint that accepts JSON data, processes it through a scikit-learn model, and returns the prediction. Some common mistakes include not handling JSON data correctly or misconfiguring the route path. Additionally, ensure that your input features match the expected format for your model.

If you were asked to handle multiple features, consider passing them as an array in the JSON request. Also, if the feature extraction process is complex, it might be better to encapsulate this logic in a separate function.

**Code:**

```python
from flask import Flask, request, jsonify
import joblib

app = Flask(__name__)

# Load the trained model (assumed you have saved it as 'model.pkl')
model_path = 'path/to/model.pkl'
model = joblib.load(model_path)

@app.route('/predict', methods=['POST'])
def predict():
    # Get JSON data from request
    json_data = request.get_json()

    # Extract features and prepare them for the model (assuming `data` is the key)
    input_features = [json_data['data']]

    # Make prediction using the loaded model
    prediction = model.predict(input_features)

    return jsonify({'prediction': prediction.tolist()})
```

**Example input / output:**

```text
# Sample Input JSON Data
{
  "data": [6.2, 3.5, 5.2, 2.4]
}

# Expected Response
{
  "prediction": [1]
}
```

