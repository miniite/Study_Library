---
type: "practice"
difficulty: "Basic"
---

# How would you handle a malformed prediction request in Flask and return a clear 400 error instead of a 500?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Error Handling](<../../../../TOOLS/Python/Flask/Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle a malformed prediction request in Flask and ensure that a 400 Bad Request status code is returned rather than a 500 Internal Server Error, we need to validate the incoming request data at the start of our API endpoint function. This involves checking if the necessary fields are present and correctly formatted before attempting any operations on them. By doing so, we can return a clear error message that helps the client understand what went wrong.

#### Triage
What is Being Probed

The interviewer is probing your understanding of how to handle errors in Flask applications, specifically focusing on HTTP status codes and request validation. They want to ensure you grasp the difference between 400 (Bad Request) and 500 (Internal Server Error), as well as your ability to return meaningful error messages that help with debugging.

#### Assumptions
The API expects a JSON payload containing specific fields.
We will use Flask to create the endpoint.
The data validation logic is implemented in a function, ensuring it can be reused across multiple endpoints if needed.

#### First Principles Solution
Define the Endpoint: Create an API endpoint that accepts POST requests with JSON data.
Validate Input: Use Python's json module or third-party libraries like marshmallow to validate the incoming request payload.
Return 400 on Malformed Data: If validation fails, return a 400 status code along with a clear error message.

Here’s how you can implement this:

PYTHON
from flask import Flask, request, jsonify

app = Flask(__name__)

# Define the expected fields in the request payload
EXPECTED_FIELDS = ['feature1', 'feature2']

def validate_request(data):
    """Validate if the required fields are present and correctly formatted."""
    # Check if all required fields are present
    missing_fields = [field for field in EXPECTED_FIELDS if field not in data]
    if missing_fields:
        return False, {"error": f"Missing fields: {', '.join(missing_fields)}"}

    # Additional validation can be added here (e.g., checking if values are integers, within range)
    # For simplicity, we assume all checks pass

    return True, {}

@app.route('/predict', methods=['POST'])
def predict():
    """Endpoint to handle prediction requests."""
    try:
        data = request.get_json()  # Get the JSON payload from the request
        is_valid, error_message = validate_request(data)

        if not is_valid:
            return jsonify(error_message), 400

        # Proceed with further processing (e.g., model prediction) using `data`
        result = "Prediction Result"  # Placeholder for actual prediction logic

        return jsonify({"result": result}), 200

    except Exception as e:
        app.logger.error(f"An error occurred: {str(e)}")
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups

In this implementation, we validate the incoming JSON payload to ensure it contains all required fields. If any field is missing or invalidly formatted, a 400 Bad Request response is returned with an appropriate error message. This approach ensures that client-side errors are caught early, providing better user experience and easier debugging.

#### Common mistakes
include not validating input at all, which can lead to internal server errors due to unhandled exceptions, or returning generic messages that do not provide useful information for troubleshooting.

For follow-ups, the interviewer might ask about handling different types of validation (e.g., data type checks), integrating more sophisticated error logging, or discussing how this solution scales with a larger number of required fields.

