---
type: "practice"
difficulty: "Deep"
---

# How would you mock a database or external API call when testing a Flask endpoint?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Testing Flask Endpoints](<../../../../TOOLS/Python/Flask/Testing_Flask_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To mock a database or external API call when testing a Flask endpoint, we need to isolate the part of the application that interacts with these dependencies so that our tests are not affected by external factors like network latency or database availability. The key idea here is using Python’s unittest.mock library, which provides tools for creating mock objects. This approach allows us to simulate the behavior of the real database or API during testing.

#### Triage

The question is asking how to create a controlled environment for testing Flask endpoints that involve external systems (like databases or APIs). The interviewer might be probing your understanding of mocking frameworks, test isolation, and best practices in writing maintainable tests. Assumptions we can make include the use of Python’s unittest.mock library.

#### First-Principles Solution
Setup Mock Objects: Create mock objects that mimic the behavior of the actual database or API.
Inject Dependencies: Pass these mocks into your Flask application code where they would normally interact with real systems.
Write Tests: Use unittest and its test runner to set up, run, and assert against your tests.
Step-by-Step Solution

Let’s consider a simple example where we want to mock a database query. We'll use the following Flask endpoint:

PYTHON
from flask import Flask

app = Flask(__name__)

def get_user(user_id):
    return User.query.get(user_id)  # This is where we would normally interact with the database

@app.route('/user/<int:user_id>')
def user_profile(user_id):
    user = get_user(user_id)
    if user:
        return f"User {user.name}'s profile"
    else:
        return "User not found", 404
Step 1: Setup Mock Object

We’ll use unittest.mock to create a mock object for the database query.

PYTHON
from unittest.mock import MagicMock, patch

# Mock User class and its methods
class MockUserClass:
    @staticmethod
    def get(user_id):
        return {"id": user_id, "name": f"Mock Name {user_id}"}

mock_db = MagicMock()
mock_db.User.return_value.get = MockUserClass.get
Step 2: Inject Dependencies

Next, we inject this mock object into our application code. This can be done by modifying the get_user function to accept a database connection as an argument.

PYTHON
def get_user(db_connection, user_id):
    return db_connection.User.get(user_id)

@app.route('/user/<int:user_id>')
def user_profile(user_id):
    with patch('flask.Flask', lambda x: app):
        db = mock_db  # Inject the mock database connection
        user = get_user(db, user_id)
        if user:
            return f"User {user['name']}'s profile"
        else:
            return "User not found", 404

# Test with mocks
from unittest.mock import patch

with patch('flask.Flask', lambda x: app), patch.object(mock_db.User, 'get'):
    response = user_profile(1)
Step 3: Write Tests

Finally, we write tests to ensure our endpoint behaves as expected when called.

PYTHON
import unittest
from flask import Flask

class TestUserProfile(unittest.TestCase):
    def setUp(self):
        self.app = app.test_client()

    def test_user_exists(self):
        response = self.app.get('/user/1')
        self.assertIn(b"Mock Name 1's profile", response.data)

    def test_user_not_found(self):
        response = self.app.get('/user/2')
        self.assertIn(b"User not found", response.data)

if __name__ == "__main__":
    unittest.main()

#### Closing Summary

By using Python’s unittest.mock library, we can effectively mock external dependencies like databases or APIs in our Flask application tests. This ensures that our tests are isolated and reliable, allowing us to focus on testing the logic of our endpoints without worrying about external systems.

#### Common Mistakes/Follow-Ups
Not Patching Correctly: Make sure you patch the correct objects and methods.
Over-Mocking: Avoid mocking too much; only mock what’s necessary for your tests.
Test Isolation: Ensure each test case is independent to avoid side effects from other tests.

**Code:**

```python
from flask import Flask, jsonify

app = Flask(__name__)

def get_user(user_id):
    return User.query.get(user_id)

@app.route('/user/<int:user_id>')
def user_profile(user_id):
    user = get_user(user_id)
    if user:
        return f"User {user.name}'s profile"
    else:
        return "User not found", 404

class MockUserClass:
    @staticmethod
    def get(user_id):
        return {"id": user_id, "name": f"Mock Name {user_id}"}

mock_db = MagicMock()
mock_db.User.return_value.get = MockUserClass.get
```

