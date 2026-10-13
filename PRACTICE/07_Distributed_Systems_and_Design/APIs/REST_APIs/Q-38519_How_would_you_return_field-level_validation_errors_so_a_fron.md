---
type: "practice"
difficulty: "Intermediate"
---

# How would you return field-level validation errors so a frontend can highlight exactly which inputs are wrong?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Status Codes](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Status_Codes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When returning field-level validation errors, we aim to provide clear and actionable feedback to the frontend developer, ensuring they can easily identify and fix the issues in user input forms. In this scenario, as an intermediate Backend SDE-2/3 engineer, you would focus on crafting a REST API that returns detailed error messages for each invalid field.

The question is probing your ability to design a clear and structured error response format that the frontend can parse and display accurately. The core concept involves understanding HTTP status codes, JSON data structures, and best practices in API design.

#### Assumptions

We are using HTTP POST requests for form submissions.
The API will return 400 Bad Request when validation fails.
Each field's error should be clearly identified by its name or ID.

Let's break this down from first principles:

HTTP Status Code: Use 400 Bad Request to indicate that the request was invalid, due to client error (like malformed JSON or incorrect data).

Error Response Structure:

Define a consistent and structured format for errors.
Include field names and specific error messages.

Field-Level Validation Example: Suppose you have a user registration form with fields username, email, and password. If the API receives invalid data, it should return an error response indicating which field(s) are problematic.

Here’s how we can implement this in code:

PYTHON
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/register', methods=['POST'])
def register_user():
    # Simulated validation logic
    data = request.get_json()

    errors = {}

    if not data['username']:
        errors['username'] = 'Username is required.'
    if '@' not in data['email']:
        errors['email'] = 'Invalid email format.'
    if len(data['password']) < 8:
        errors['password'] = 'Password must be at least 8 characters long.'

    # If there are no errors, process the request
    if not errors:
        return jsonify({'message': 'User registered successfully.'})

    # Return error response with field-level validation messages
    return jsonify({'errors': errors}), 400

if __name__ == '__main__':
    app.run(debug=True)

This code snippet demonstrates a simple Flask API that validates user input and returns specific error messages for each invalid field. The frontend can then use this information to highlight the problematic fields.

#### Closing Summary
By structuring our response as a JSON object with errors containing key-value pairs of field names and their corresponding validation errors, we ensure the frontend can display accurate feedback. This approach leverages clear HTTP status codes (400 Bad Request) and structured error messages for effective communication between backend and frontend.

#### Common Mistakes/Follow-Ups

Forgetting to include all relevant fields in the errors object.
Not returning a 400 status code when validation fails.
Using generic error messages instead of specific field-level feedback.
Failing to test edge cases, such as missing or incorrect data types.

