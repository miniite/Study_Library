---
type: "practice"
difficulty: "Intermediate"
---

# How would you build a Flask endpoint that returns feature importances or a simple explanation for a prediction?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Loading Models Efficiently](<../../../../TOOLS/Python/Flask/Loading_Models_Efficiently.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a Flask endpoint that returns feature importances or a simple explanation for a prediction, we need to create an API that accepts input data, processes it through a machine learning model, and then generates and returns either the feature importances or an explanation of the prediction. This involves setting up a Flask application, defining routes, handling input data, making predictions with a trained model, and constructing the response.

The key idea is to leverage Flask for web development and integrate it with our ML model. We'll handle different types of requests (e.g., POST) where we receive input features, process them using the model, and then return either feature importances or an explanation.

**Code:**

```python
from flask import Flask, request, jsonify
import joblib

app = Flask(__name__)

# Load the trained machine learning model
model = joblib.load('trained_model.pkl')

@app.route('/predict', methods=['POST'])
def predict():
    # Get input data from the request
    input_data = request.json

    # Ensure the input has the expected format and features
    if not isinstance(input_data, dict) or 'features' not in input_data:
        return jsonify({'error': 'Invalid input format'}), 400

    # Extract the features
    features = input_data['features']

    # Make a prediction using the model
    prediction = model.predict([features])

    # Generate feature importances or explanation
    if request.args.get('importance'):
        # Compute feature importance (assuming model provides this)
        importances = model.feature_importances_
        response = {'prediction': float(prediction[0]), 'feature_importances': importances.tolist()}
    else:
        response = {'prediction': float(prediction[0])}

    return jsonify(response)

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```

Example input JSON:

JSON
Copy
{
  "features": [0.5, 1.2, -0.3, 0.8]
}

Response when importance parameter is not provided:

JSON
Copy
{
  "prediction": 0.75
}

Response when importance parameter is provided:

JSON
Copy
{
  "prediction": 0.75,
  "feature_importances": [0.2, 0.4, 0.1, 0.3]
}
```

**Flow:**

1. Yes
2. No
3. Receive POST request
4. Is 'importance' parameterprovided?
5. Generate feature importances
6. Make prediction only
7. Return response withprediction only

