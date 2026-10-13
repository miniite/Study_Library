---
type: "practice"
difficulty: "Intermediate"
---

# How should passwords be stored when a user registers in a Flask app?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Authentication in Flask](<../../../../TOOLS/Python/Flask/Authentication_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling password storage in a Flask application, security is paramount. A common approach involves hashing the password before storing it. This ensures that even if the database is compromised, the actual passwords are not exposed. For this task, we'll discuss how to implement secure password storage using Flask and Python.

First, let's clarify what the interviewer might be probing: they want to ensure you understand best practices for handling sensitive user data, specifically focusing on password security. You should demonstrate knowledge of hashing algorithms, environment configuration, and security libraries.

#### Assumptions

The application is a typical web app that uses Flask.
Users provide passwords in clear text during registration.
We are storing the hashed version of the password securely.
The Flask application runs on a secure server with proper security configurations (e.g., HTTPS).

Now, let's walk through the steps from first principles to implement this:

Choose a Hashing Algorithm: Passwords should be salted and hashed using a strong algorithm like bcrypt, which is designed for password hashing.
Install Required Libraries: You need to install the flask-bcrypt library, which provides bcrypt functionality.
Configure Flask App: Set up the Flask application to use the bcrypt library securely.
Hash Passwords During Registration: Ensure that passwords are hashed and stored correctly.

Here’s a step-by-step solution:

Install Dependencies:

SH
pip install flask flask-bcrypt

Set Up Flask App:

PYTHON
from flask import Flask, request
from flask_bcrypt import Bcrypt

app = Flask(__name__)
bcrypt = Bcrypt(app)

# Example user model and database setup (using in-memory SQLite for simplicity)
users_db = {}

@app.route('/register', methods=['POST'])
def register():
    username = request.form['username']
    password = request.form['password']

    # Generate a salt
    hashed_password = bcrypt.generate_password_hash(password).decode('utf-8')

    # Store the user in the database with their hashed password
    users_db[username] = {'hashed_password': hashed_password}

    return f"User {username} registered successfully."

Run and Test the Application:

SH
flask run

You can use curl or a web client to test the registration endpoint.

Handle Password Verification: When a user logs in, you should hash the provided password using the same method and compare it with the stored hashed password.

Secure Environment Variables: Use environment variables for sensitive data like secret keys used by Flask-Bcrypt, ensuring these are not hard-coded in your source code.

By following these steps, we ensure that passwords are securely stored and protected against potential security breaches.

#### Closing summary
To store passwords securely in a Flask application, use bcrypt to hash the password before storing it. This approach ensures data integrity and confidentiality. Remember to configure your application properly and handle sensitive information carefully.

