---
type: "practice"
difficulty: "Basic"
---

# Where in a Flask app would you load a pickled model so it is shared across requests, and how do you make it globally accessible?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Flask vs Dedicated Model Servers](<../../../../TOOLS/Python/Flask/Flask_vs_Dedicated_Model_Servers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, let's first understand what we're dealing with. We need to load a pickled machine learning model in a Flask application so that it can be shared across multiple requests and made globally accessible. This means that the model should be loaded once when the application starts and then used by different routes without being reloaded for each request.

#### Triaging the Question

The core of this question is about how to initialize and make a global resource (in this case, a pickled model) available across multiple requests in a Flask application. We need to ensure that the model loading process is efficient and only happens once when the server starts up.

#### First-Principles Solution
Model Loading: Load the pickled model during the initialization phase of the Flask app.
Global Scope: Make sure the model object is available globally within the application context so that it can be accessed by different routes and functions without needing to reload it for each request.
Step-by-Step Solution

Define the Application Structure:

A typical Flask application has a structure where the __init__.py file initializes the app.

Loading the Model:

We can load the model in the __init__.py or another initialization script, which will be run once when the server starts.

Making it Globally Accessible:

Use a global variable within the Flask application context to hold the model object.

#### Example
Implementation

Here's how you might implement this:

PYTHON
# app/__init__.py
from flask import Flask
import joblib

def create_app():
    app = Flask(__name__)

    # Load pickled model during initialization
    model = joblib.load('path/to/your/model.pkl')

    @app.route('/predict', methods=['POST'])
    def predict():
        # Use the global model object here
        data = request.get_json()
        prediction = model.predict([data])
        return jsonify(prediction.tolist())

    return app

if __name__ == '__main__':
    app = create_app()
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups

In this example, the model is loaded in the create_app() function during initialization. The model object is stored as a global variable within this scope, making it accessible to any route defined in the application. This ensures that the model is only loaded once and can be used by multiple routes.

#### Common Mistakes
Re-loading the Model for Each Request: This could happen if you load the model inside each route instead of initializing it globally.
Ignoring Application Context: If not using Flask's global context properly, the model might not be accessible from different parts of your application.
Triaging Follow-Ups

#### Performance Considerations
Ensure that loading large models during initialization doesn't slow down server startup significantly.
Security: Be cautious about where you store sensitive data like model files and ensure proper access control.

