---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement 'remember me' functionality using Flask-Login?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Authentication in Flask](<../../../../TOOLS/Python/Flask/Authentication_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement "remember me" functionality in a Flask application, we need to manage user sessions effectively. The core idea is to use Flask-Login and set session cookies that persist across browser closures. Here’s how you can approach this from first principles.

Firstly, the question asks us to implement "remember me" functionality using Flask-Login, which means we will leverage its built-in mechanisms for managing user sessions and authentication states.

#### Triage

The interviewer is probing your understanding of session management, cookie handling, and secure storage. You need to demonstrate knowledge about how cookies can be used to store persistent data (like a unique token) that identifies the user across sessions. The key assumptions are:

Users must have an account already set up.
Security considerations must be addressed to prevent unauthorized access.

#### First-Principles Solution

Setup Flask and Flask-Login:

Ensure you have Flask and Flask-Login installed in your project.

Create User Model:

PYTHON
from flask_login import UserMixin

class User(UserMixin):
    def __init__(self, id, username):
        self.id = id
        self.username = username

Initialize Flask and Flask-Login:

Initialize a Flask app.
Initialize Flask-Login and configure it to handle sessions.

Implement User Loader:

Define how the user is loaded from an ID.

Set Up Remember Me Functionality:

Use remember parameter in the login function to set a longer-lived session cookie.
Handle cookie persistence, including secure and HTTP-only flags for better security.

Protect Routes with Login Required Decorator:

Ensure critical routes are protected by checking if the user is logged in.

#### Implementation

Let's walk through each step:

PYTHON
from flask import Flask, request, redirect, url_for, session
from flask_login import LoginManager, UserMixin, login_user, logout_user, current_user

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your_secret_key'
login_manager = LoginManager()
login_manager.init_app(app)

# Sample user data (in practice, this would come from a database)
users = {
    1: {"id": 1, "username": "user1"},
    2: {"id": 2, "username": "user2"}
}

class User(UserMixin):
    def __init__(self, id, username):
        self.id = id
        self.username = username

@login_manager.user_loader
def load_user(user_id):
    return User(**users[int(user_id)])

@app.route('/login', methods=['POST'])
def login():
    data = request.form
    user = users.get(int(data['id']))
    if user:
        # Set the remember parameter to True for a longer-lived session cookie
        login_user(User(user['id'], user['username']), remember=True)
        return redirect(url_for('protected'))
    else:
        return "User not found", 401

@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('login'))

@login_manager.unauthorized_handler
def unauthorized():
    return "You must be logged in to access this page.", 403

@app.route('/protected')
@login_required
def protected():
    if current_user.is_authenticated:
        return f"Hello, {current_user.username}! You are logged in."
    else:
        return redirect(url_for('login'))

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By using Flask-Login's remember parameter, we can set a longer-lived session cookie that persists across browser sessions. This requires careful consideration of security to prevent unauthorized access.

#### Common Mistakes
Forgetting to use the remember=True flag when logging in users.
Not handling user authentication and authorization correctly.
Neglecting to securely store sensitive information like secret keys.
Further Discussion

The interviewer might ask about implementing additional security measures, such as using HTTPS or setting secure flags on cookies. You could also discuss how Flask-Session can be used for more complex session management scenarios.

