---
type: "practice"
difficulty: "Intermediate"
---

# How do you write a unit test for a Flask /predict endpoint that mocks the model so the test does not depend on the real model?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Testing Flask Endpoints](<../../../../TOOLS/Python/Flask/Testing_Flask_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a unit test for a Flask /predict endpoint, we need to ensure our tests are isolated from any external dependencies such as the actual machine learning model. By mocking the model, we can focus on testing the behavior of the API itself, ensuring that it correctly handles requests and returns appropriate responses. This approach helps in maintaining clean and reliable test cases.

#### Triaging the Question

The core of this question is to understand how to write a unit test for a Flask endpoint while isolating it from real model dependencies. The key points include:

Mocking: We need to mock the ML model's behavior.
Test Isolation: Ensure that tests do not rely on external state or services.
Test Cases: Create specific test cases that cover different scenarios (e.g., valid input, invalid input).

#### First-Principles Solution
Set Up Test Environment:
Install necessary packages: pytest, unittest, and a library for mocking like unittest.mock.
Define the Flask Endpoint:
Create or identify an existing /predict endpoint that takes input data, passes it to the model, and returns a response.
Mock the Model:
Use Python's mock library to replace the actual model with a mock object.
Write Test Cases:
Define test cases for different scenarios (e.g., correct input, edge cases).
Step-by-Step Solution
1. Set Up Your Environment

First, ensure you have the necessary packages installed. You can use pip to install them if they are not already available.

BASH
pip install flask pytest unittest.mock
2. Define the Flask App and Endpoint

Here is a simple example of how your Flask app might look:

PYTHON
# app.py

from flask import Flask, request, jsonify

app = Flask(__name__)

def predict(input_data):
    # This is where you would have your actual model code.
    return {"prediction": "mocked_prediction"}

@app.route('/predict', methods=['POST'])
def predict_endpoint():
    input_data = request.get_json()
    result = predict(input_data)
    return jsonify(result), 200
3. Write the Unit Test

Now, write a unit test using pytest and unittest.mock. This will involve mocking the model function.

PYTHON
# tests/test_predict.py

from app import app, predict
import unittest
from unittest.mock import patch
import json

class TestPredictEndpoint(unittest.TestCase):
    def setUp(self):
        self.app = app.test_client()

    @patch('app.predict')
    def test_predict_endpoint_valid_input(self, mock_predict):
        # Mock the model's response
        mock_predict.return_value = {"prediction": "valid_prediction"}

        # Send a POST request with valid input data
        response = self.app.post('/predict', json={"input": "data"})

        # Check that the response is correct and the status code is 200
        self.assertEqual(response.status_code, 200)
        self.assertIn("prediction", json.loads(response.data))

    @patch('app.predict')
    def test_predict_endpoint_invalid_input(self, mock_predict):
        # Mock the model's response for invalid input
        mock_predict.return_value = {"error": "invalid_input"}

        # Send a POST request with invalid input data
        response = self.app.post('/predict', json={"input": ""})

        # Check that the response is correct and the status code is 400
        self.assertEqual(response.status_code, 400)
        self.assertIn("error", json.loads(response.data))

if __name__ == "__main__":
    unittest.main()

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this example, we've created a test suite that includes two test cases: one for valid input and another for invalid input. We used the patch decorator from the unittest.mock module to replace the actual predict function with a mock object. This allows us to control its behavior during testing without relying on an external model.

#### Common Mistakes
Not Mocking: Forgetting to use mocks can lead to tests that are tightly coupled and harder to maintain.
Lack of Test Coverage: Not covering all possible scenarios (e.g., edge cases, unexpected input formats).
Overly Complex Mocks: Using overly complex or redundant mock configurations.

#### Follow-Ups

#### Cons
ider adding more test cases for different types of inputs.
Validate the integration between the Flask app and your database if applicable.
Explore other testing strategies like integration tests to ensure end-to-end functionality.

**Code:**

```
app.py

from flask import Flask, request, jsonify

app = Flask(name)

def predict(input_data): # This is where you would have your actual model code. return {"prediction": "mocked_prediction"}

@app.route('/predict', methods=['POST']) def predict_endpoint(): input_data = request.get_json() result = predict(input_data) return jsonify(result), 200

CODE
Copy

```python
# tests/test_predict.py

from app import app, predict
import unittest
from unittest.mock import patch
import json

class TestPredictEndpoint(unittest.TestCase):
    def setUp(self):
        self.app = app.test_client()

    @patch('app.predict')
    def test_predict_endpoint_valid_input(self, mock_predict):
        # Mock the model's response
        mock_predict.return_value = {"prediction": "valid_prediction"}

        # Send a POST request with valid input data
        response = self.app.post('/predict', json={"input": "data"})

        # Check that the response is correct and the status code is 200
        self.assertEqual(response.status_code, 200)
        self.assertIn("prediction", json.loads(response.data))

    @patch('app.predict')
    def test_predict_endpoint_invalid_input(self, mock_predict):
        # Mock the model's response for invalid input
        mock_predict.return_value = {"error": "invalid_input"}

        # Send a POST request with invalid input data
        response = self.app.post('/predict', json={"input": ""})

        # Check that the response is correct and the status code is 400
        self.assertEqual(response.status_code, 400)
        self.assertIn("error", json.loads(response.data))

if __name__ == "__main__":
    unittest.main()
```

