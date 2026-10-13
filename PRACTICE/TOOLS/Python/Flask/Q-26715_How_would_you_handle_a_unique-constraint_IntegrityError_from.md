---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle a unique-constraint IntegrityError from the database in a Flask registration endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Error Handling](<../../../../TOOLS/Python/Flask/Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling a UniqueConstraint IntegrityError in a Flask registration endpoint, the key goal is to provide clear and actionable feedback to the user while ensuring security measures are in place. The core idea here is to intercept the error, determine its cause, and return an appropriate response.

#### Triage

The question asks how to handle an IntegrityError related to a unique constraint (such as a unique email or username) in a Flask registration endpoint. This involves understanding that when the database rejects a row because of a unique constraint violation, this exception must be caught and handled gracefully. The interviewer is likely probing your ability to manage errors, user experience design, and security practices.

#### Assumptions
You are using SQLAlchemy for ORM with Flask.
The registration endpoint expects an email and password from the client.
The database has a unique constraint on the email column in the users table.

#### First-Principles Solution
Intercepting the Error:
Use a try-except block to catch the IntegrityError.
Determining the Cause:
Check which unique constraint was violated.

#### Cons
tructing the Response:
Provide clear, non-technical error messages.
Security Considerations:
Avoid exposing sensitive information such as email addresses in error messages.

Here’s a step-by-step solution:

Define the registration route in your Flask application:

PYTHON
from flask import Flask, request, jsonify
from sqlalchemy.exc import IntegrityError
from models import User  # Assuming you have a User model with SQLAlchemy

app = Flask(__name__)

@app.route('/register', methods=['POST'])
def register():
    try:
        email = request.json.get('email')
        password = request.json.get('password')

        new_user = User(email=email, password=password)  # Simplified for example
        db.session.add(new_user)
        db.session.commit()
    except IntegrityError as e:
        if 'email' in str(e):  # Assuming the unique constraint is on email
            return jsonify({"error": "Email already exists. Please use a different email."}), 409
        else:
            raise  # Re-raise unexpected errors for debugging

if __name__ == '__main__':
    app.run(debug=True)

Explanation:

The try-except block ensures that the code within it is executed, and any exceptions are caught.
By checking the exception message string, you can determine which constraint was violated (in this case, 'email').
A 409 Conflict status is returned for unique email errors, providing a clear HTTP error response.

Security Considerations:

Avoid exposing sensitive information such as password in the response.
Ensure that you handle other potential constraints and edge cases similarly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, handling IntegrityError requires catching exceptions correctly, determining their cause, and returning appropriate HTTP responses. A key mistake would be not providing clear error messages to users or exposing sensitive information in the response. Follow-ups might include discussing how to handle other types of database errors, implementing retries for temporary issues, and integrating with a front-end for enhanced user experience.

**Code:**

```python
from flask import Flask, request, jsonify
from sqlalchemy.exc import IntegrityError
from models import User  # Assuming you have a User model with SQLAlchemy

app = Flask(__name__)

@app.route('/register', methods=['POST'])
def register():
    try:
        email = request.json.get('email')
        password = request.json.get('password')

        new_user = User(email=email, password=password)  # Simplified for example
        db.session.add(new_user)
        db.session.commit()
    except IntegrityError as e:
        if 'email' in str(e):  # Assuming the unique constraint is on email
            return jsonify({"error": "Email already exists. Please use a different email."}), 409
        else:
            raise  # Re-raise unexpected errors for debugging

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```text
Input:
POST /register
{
    "email": "example@example.com",
    "password": "securepassword"
}

Response:
HTTP 201 Created

Input:
POST /register
{
    "email": "alreadyused@example.com",
    "password": "securepassword"
}

Response:
HTTP 409 Conflict
{
    "error": "Email already exists. Please use a different email."
}
```

