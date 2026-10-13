---
type: "practice"
difficulty: "Intermediate"
---

# What does db.session.rollback() do and when should you call it in a Flask request handler?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Flask-SQLAlchemy](<../../../../TOOLS/Python/Flask/Flask-SQLAlchemy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage database transactions effectively, understanding db.session.rollback() is crucial for intermediate-level developers working with Flask applications using SQLAlchemy as the ORM. This method undoes all changes made to the session since it was created or the last commit/rollback occurred. You should call it in a request handler when you want to revert any database changes that might have been made, typically because an error has occurred.

#### Triage

The question is asking about the functionality and appropriate use cases of db.session.rollback() within Flask applications. The interviewer is probing your understanding of transaction management and error handling mechanisms in Flask-SQLAlchemy.

#### Assumptions
You are familiar with SQLAlchemy's session management.
You understand basic database transactions and their importance in ensuring data integrity.

#### Solution

Let’s break down the step-by-step process:

Context Setting: In a typical Flask application, you use db.session to interact with your database through SQLAlchemy. The session acts as a bridge between the ORM and the actual database connection.
Transaction Management: When you modify data using db.session, these changes are stored in memory but not yet committed to the database. If everything goes well, you commit the session (session.commit()), making all changes permanent.
Rollback Mechanism: However, if an error occurs (like a validation failure or any exception) before committing, those changes need to be undone without affecting the application's state or breaking existing transactions.
When to Call db.session.rollback()

You should call session.rollback() when you want to undo all database operations that were performed since the last commit or rollback. This is typically done in the middle of a request handler if an error occurs and you don't wish to proceed with committing changes.

#### Example Scenario

Suppose you have a Flask route for creating a new user:

PYTHON
from flask import Flask, jsonify
from sqlalchemy.exc import SQLAlchemyError

app = Flask(__name__)
db.init_app(app)  # Assuming db is your SQLAlchemy instance

@app.route('/create-user', methods=['POST'])
def create_user():
    try:
        name = request.json['name']
        email = request.json['email']

        user = User(name=name, email=email)
        db.session.add(user)  # Add new user to the session
        db.session.commit()   # Commit changes

        return jsonify({"message": "User created successfully"}), 201
    except SQLAlchemyError as e:
        db.session.rollback()  # Rollback in case of error
        print(f"An exception occurred: {e}")
        return jsonify({"error": str(e)}), 400

#### Closing Summary
and Common Mistakes/Follow-Ups

db.session.rollback() is a powerful tool for managing transactions, ensuring that any changes made during a request can be undone if an error occurs. It’s essential to use it judiciously in your Flask applications to maintain data integrity.

#### Common Mistakes

Forgetting to call rollback() when an exception occurs, leading to half-applied database operations.
Confusing session.rollback() with commit(), which can lead to inconsistent states if not used properly.

#### Follow-Ups

How does Flask handle rollback in a context other than the request handler (e.g., background tasks)?
What are some best practices for managing sessions and transactions in a multi-threaded environment?

**Code:**

```python
from flask import Flask, jsonify
from sqlalchemy.exc import SQLAlchemyError

app = Flask(__name__)
db.init_app(app)  # Assuming db is your SQLAlchemy instance

@app.route('/create-user', methods=['POST'])
def create_user():
    try:
        name = request.json['name']
        email = request.json['email']

        user = User(name=name, email=email)
        db.session.add(user)  # Add new user to the session
        db.session.commit()   # Commit changes

        return jsonify({"message": "User created successfully"}), 201
    except SQLAlchemyError as e:
        db.session.rollback()  # Rollback in case of error
        print(f"An exception occurred: {e}")
        return jsonify({"error": str(e)}), 400
```

**Example input / output:**

```text
Input Request (POST /create-user):
{
    "name": "John Doe",
    "email": "john.doe@example.com"
}
Output Response:
201 - {"message": "User created successfully"}
Exceptional Scenario:
Input Request (POST /create-user) with missing data field:
{
    "name": "Jane Doe"
}
Output Response (with error handling):
400 - {"error": "Validation Error: Missing 'email' in request data"}
```

**Flow:**

1. No
2. Yes
3. Start Request
4. Error Occurred?
5. Commit Session
6. Return Success Response
7. Rollback Session
8. Return Error Response

