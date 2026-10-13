---
type: "practice"
difficulty: "Intermediate"
---

# How do you hash and verify passwords using werkzeug.security's generate_password_hash and check_password_hash?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ CORS](<../../../../TOOLS/Python/Flask/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely handle password hashing and verification in a Flask application, we use werkzeug.security's generate_password_hash and check_password_hash. These functions provide a robust way to store and check passwords without needing to manage the hash algorithms manually.

#### Triage

The question is asking for how to use these functions effectively within a Flask application. The interviewer will be probing your understanding of password security best practices, specifically hashing and verifying passwords securely. Key assumptions include:

You are familiar with basic Flask setup.
You understand the importance of secure password storage.
You know how to integrate external libraries in Flask.

#### First-principles Solution
Install Werkzeug: If not already installed, you need to add werkzeug to your project dependencies via pip: pip install werkzeug.
Hashing Passwords:
When a user signs up or changes their password, use generate_password_hash() to create a secure hash.
Store this hash securely in the database.
Verifying Passwords:
When authenticating a user, retrieve the stored hash from the database and pass it along with the entered password to check_password_hash().

#### Example
Code: Here’s how you can implement these steps in your Flask application.

#### Example
Code
PYTHON
from flask import Flask, request
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)

# Simulate a database entry
users_db = {
    'user1': generate_password_hash('password1'),
    'user2': generate_password_hash('password2')
}

@app.route('/register', methods=['POST'])
def register():
    username = request.form.get('username')
    password = request.form.get('password')

    # Hash the password
    hashed_password = generate_password_hash(password)
    users_db[username] = hashed_password

    return f"User {username} registered successfully."

@app.route('/login', methods=['POST'])
def login():
    username = request.form.get('username')
    password = request.form.get('password')

    # Check if the user exists
    if username in users_db:
        stored_hash = users_db[username]

        # Verify the password
        if check_password_hash(stored_hash, password):
            return f"Login successful for {username}."
        else:
            return "Incorrect password."
    else:
        return "User not found."

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Using werkzeug.security ensures secure password handling by automatically managing the hashing algorithm and salt.

#### Common Mistakes
Storing plain text passwords: This is a significant security risk.
Failing to update hashes on password changes: Passwords should always be rehashed when changed.
Not using HTTPS: Transmitting passwords over unsecured connections can compromise them.
Additional Follow-Up
How would you integrate this with a production database?
What about handling password resets and recovery?
Are there any other best practices for password management in web applications?

