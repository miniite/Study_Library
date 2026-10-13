---
type: "practice"
difficulty: "Basic"
---

# How would you add an /health endpoint that also confirms the ML model was loaded successfully in a Flask service?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Testing Flask Endpoints](<../../../../TOOLS/Python/Flask/Testing_Flask_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add an /health endpoint to confirm that the machine learning (ML) model is loaded successfully in a Flask service, we need to ensure two things: first, the server is up and running; second, the ML model has been correctly loaded. Let's break this down step by step.

#### Triaging the Question

The question asks us to implement an /health endpoint that checks if the Flask application is operational and if the ML model is ready for use. The interviewer might be probing your understanding of Flask’s routing system, how to handle HTTP requests, and basic error handling. We will assume the following:

The Flask service has been deployed.
The ML model is stored as a file or object that can be loaded.
You have a way to load the ML model (e.g., from disk or a database).

#### First-Principles Solution
Define the /health endpoint: This will handle HTTP GET requests and return a status code indicating whether the service is healthy.
Check Flask’s readiness: Ensure that the server is running by checking its status.
Verify ML model load: Load the ML model and check if it was loaded successfully.
Step-by-Step Solution

Set Up the Flask Application:

PYTHON
from flask import Flask, jsonify
import joblib  # Example library for loading models

app = Flask(__name__)

Load the ML Model:

PYTHON
model_path = 'path/to/your/model.pkl'
try:
    model = joblib.load(model_path)
    print("Model loaded successfully.")
except Exception as e:
    print(f"Failed to load model: {e}")
    exit(1)  # Exit the application if model loading fails

Define the /health Endpoint:

PYTHON
@app.route('/health', methods=['GET'])
def health():
    return jsonify({
        "status": "healthy",
        "model_loaded": model is not None,
        "message": "Model loaded successfully." if model else "Model load failed."
    })

Run the Flask Application:

PYTHON
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we've ensured that both the server status and the ML model load are checked before returning a health check response. The /health endpoint will provide useful information about the application’s state, making it easier to diagnose issues.

#### Common Mistakes

Forgetting to handle exceptions when loading the ML model.
Not checking if the ML model is loaded successfully before returning the health status.
Running the Flask app without proper error handling mechanisms in place.

#### Follow-Ups

Discussing how you would log detailed information about failures.
Exploring alternatives for model storage and retrieval (e.g., using a database).

#### Cons
idering security aspects, such as protecting sensitive model files.

