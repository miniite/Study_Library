---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd add request/response validation and schema-driven OpenAPI docs to an existing Flask app that wasn't built with it, incrementally.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/Python_Language/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add request/response validation and generate schema-driven OpenAPI documentation in a Flask application, we'll follow a structured approach. First, let’s quickly summarize what we’re aiming to achieve: We need to ensure that incoming requests adhere to specific schemas, process the data correctly, and return responses with appropriate structures. Additionally, we want to leverage the OpenAPI specification (formerly known as Swagger) to document our API.

#### Triaging the Question

The interviewer is likely probing your understanding of:

Request/Response Validation: Ensuring that incoming requests are structured properly.
OpenAPI Documentation: Generating and maintaining API documentation using standard formats like OpenAPI.
Incremental Changes: Modifying an existing application without disrupting its functionality.

#### Assumptions
The Flask app already serves some endpoints but lacks validation and comprehensive documentation.
We have access to the source code, necessary development environments (Python, pip), and possibly a backend database for storing or querying data.
Our goal is to add these features incrementally without major refactoring of existing code.
Step-by-Step Solution from First Principles
1. Install Required Packages

First, we need to install the Flask-Swagger package for generating OpenAPI documentation and a validation library like marshmallow. We can use Flask extensions that handle both:

BASH
pip install flask-marshmallow marshmallow-sqlalchemy flask-swagger
2. Define Schemas

We start by defining schemas using Marshmallow to validate incoming requests. For example, if we have an endpoint for creating a user with fields like name, email, and password:

PYTHON
from flask_marshmallow import Marshmallow
from marshmallow import Schema, fields

ma = Marshmallow()

class UserSchema(Schema):
    name = fields.String(required=True)
    email = fields.Email(required=True)
    password = fields.String(required=True)

user_schema = UserSchema()
users_schema = UserSchema(many=True)
3. Integrate Validation in Endpoints

Next, we integrate these schemas into the Flask endpoints to validate incoming requests:

PYTHON
from flask import request
from flask_marshmallow import Marshmallow

app = Flask(__name__)
ma.init_app(app)

@app.route('/user', methods=['POST'])
def create_user():
    user_input = request.get_json()
    user = UserSchema().load(user_input)
    if not user.is_valid:
        return user.messages, 400

    # Process valid data
    ...

    return user.dump(), 201
4. Generate OpenAPI Documentation

We can now use flask-swagger to generate and serve the documentation:

PYTHON
from flask_swagger import swagger

@app.route('/api/docs', methods=['GET'])
def spec():
    swag = swagger(app)
    swag['info']['version'] = "1.0"
    swag['info']['title'] = "User API"

    return jsonify(swag)
5. Serve and Test

Finally, we run the Flask application to serve both our API and the documentation:

PYTHON
if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary

By following this incremental approach, we ensure that our existing Flask application is updated with robust request/response validation and comprehensive OpenAPI documentation. This not only improves the quality of our APIs but also makes them more maintainable and user-friendly.

#### Common Mistakes/Follow-Ups
Failing to Validate All Requests: Ensure every endpoint has proper validation.
Over-complicating Schemas: Keep schemas simple and focused on necessary fields.
Ignoring OpenAPI Best Practices: Use consistent naming conventions, versioning, etc., in your documentation.

Would you like me to elaborate further on any part of the solution?

**Comparison:**

| Schema Field | Type     | Required |
|--------------|----------|----------|
| name         | String   | Yes      |
| email        | Email    | Yes      |
| password     | String   | Yes      |

**Code:**

```python
from flask import Flask, request, jsonify
from flask_marshmallow import Marshmallow
from marshmallow import Schema, fields

app = Flask(__name__)
ma = Marshmallow(app)

class UserSchema(Schema):
    name = fields.String(required=True)
    email = fields.Email(required=True)
    password = fields.String(required=True)

user_schema = UserSchema()
users_schema = UserSchema(many=True)

@app.route('/user', methods=['POST'])
def create_user():
    user_input = request.get_json()
    user = user_schema.load(user_input)
    if not user.is_valid:
        return user.messages, 400

    # Process valid data
    ...

    return jsonify(user.dump()), 201

@app.route('/api/docs', methods=['GET'])
def spec():
    swag = swagger(app)
    swag['info']['version'] = "1.0"
    swag['info']['title'] = "User API"

    return jsonify(swag)

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```text
Input:
POST /user HTTP/1.1
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "password": "password123"
}

Response (Status: 201):
{
  "name": "John Doe",
  "email": "john.doe@example.com"
}
```

**Data:**

```json
{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "password": "password123"
}
```

**Flow:**

1. Start
2. Define Schemas
3. Integrate Validation inEndpoints
4. Generate OpenAPIDocumentation
5. Serve API and Docs
6. End

