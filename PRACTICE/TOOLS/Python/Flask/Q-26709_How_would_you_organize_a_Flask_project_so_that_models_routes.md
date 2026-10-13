---
type: "practice"
difficulty: "Deep"
---

# How would you organize a Flask project so that models, routes, and services are cleanly separated?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Flask vs Dedicated Model Servers](<../../../../TOOLS/Python/Flask/Flask_vs_Dedicated_Model_Servers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When organizing a Flask project with clean separation of concerns between models, routes, and services, the goal is to create a maintainable, scalable, and easy-to-understand codebase. This involves structuring your application so that each component has well-defined responsibilities.

#### Triage

The question aims to ensure that the project is organized in such a way that it adheres to the principles of separation of concerns, making the application modular and easier to manage. Key assumptions include:

The project uses Flask as the web framework.
Models handle data storage and manipulation.
Routes define HTTP endpoints and interact with services.
Services encapsulate business logic.

#### First-principles Solution

To achieve clean separation, we can organize our Flask project into three main directories: models, routes, and services. Additionally, a fourth directory for shared utilities or configurations can be useful. The structure should look like this:

CODE
my_flask_app/
│
├── app.py  # Main application file
├── config.py  # Configuration settings
├── models/  # Contains SQLAlchemy models
│   ├── __init__.py
│   └── user.py  # Example model for a User entity
├── routes/  # Flask views and route handlers
│   ├── __init__.py
│   └── user_views.py  # Example view functions for the User resource
├── services/  # Business logic encapsulated in services
│   ├── __init__.py
│   └── user_service.py  # Example service for User-related operations
└── utils/  # Utility files and shared code
    ├── __init__.py
    └── helpers.py  # Example utility functions
Step-by-Step Breakdown

Models Directory:

Contains SQLAlchemy models that represent the database schema.
Each model file should have a corresponding __init__.py to initialize the model classes.

Routes Directory:

Houses view functions and route definitions.
Each route file (e.g., user_views.py) will define specific endpoints and their corresponding handler functions.
__init__.py can be used for initialization or any utility functions related to routes.

Services Directory:

Contains business logic encapsulated in service classes.
These services interact with models (for data fetching or modification) and handle complex operations.
A good practice is to have a __init__.py that initializes the services.

Utils Directory:

For shared utilities, configurations, or helper functions that can be reused across different parts of the application.
This directory ensures that common functionalities are centralized and easily accessible.

#### Implementation
Example

Here’s an example implementation for a user_service.py file:

PYTHON
# services/user_service.py

from app import db
from models.user import User  # Assuming we have a User model in models/user.py

class UserService:
    def create_user(self, user_data):
        new_user = User(**user_data)
        db.session.add(new_user)
        db.session.commit()
        return new_user

    def get_user_by_id(self, user_id):
        return User.query.get(user_id)

And here’s how a route might interact with this service in routes/user_views.py:

PYTHON
# routes/user_views.py

from flask import Blueprint, request
from services.user_service import UserService  # Import the service from the services directory
user_bp = Blueprint('users', __name__)
user_service = UserService()  # Initialize the service at the module level

@user_bp.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = user_service.get_user_by_id(user_id)
    if not user:
        return {'error': 'User not found'}, 404
    return user.to_dict()

#### Closing Summary
+ Common Mistakes/Follow-Ups

By structuring the project this way, we ensure that models, routes, and services are clearly separated, making the application easier to maintain. Common mistakes include:

Overlapping responsibilities between different components.
Not using initialization files (__init__.py) effectively, leading to redundant imports or missing initializations.

#### Follow-ups
might involve discussing testing strategies (unit tests for each component), deployment practices, and scalability considerations when dealing with a larger project.

**Comparison:**

| Component | Responsibility |
|-----------|----------------|
| Models    | Data storage and manipulation |
| Routes    | Define HTTP endpoints |
| Services  | Encapsulate business logic |
| Utils     | Shared utilities and configurations |

**Code:**

```
Example of a model in models/user.py

from app import db

class User(db.Model): id = db.Column(db.Integer, primary_key=True) username = db.Column(db.String(80), unique=True, nullable=False) email = db.Column(db.String(120), unique=True, nullable=False)

CODE
Copy
def __repr__(self):
    return f"<User {self.username}>"

@property
def to_dict(self):
    return {
        'id': self.id,
        'username': self.username,
        'email': self.email
    }
CODE
Copy

```python
# Example of a service in services/user_service.py

from app import db
from models.user import User  # Assuming we have a User model in models/user.py

class UserService:
    def create_user(self, user_data):
        new_user = User(**user_data)
        db.session.add(new_user)
        db.session.commit()
        return new_user

    def get_user_by_id(self, user_id):
        return User.query.get(user_id)
PYTHON
Copy
# Example of a route in routes/user_views.py

from flask import Blueprint, request
from services.user_service import UserService  # Import the service from the services directory
user_bp = Blueprint('users', __name__)
user_service = UserService()  # Initialize the service at the module level

@user_bp.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = user_service.get_user_by_id(user_id)
    if not user:
        return {'error': 'User not found'}, 404
    return user.to_dict()
```

**Example input / output:**

```text
Input:
GET /users/1

Output:
{
  "id": 1,
  "username": "john_doe",
  "email": "johndoe@example.com"
}

Error Case (User not found):
GET /users/999

Output:
{
  "error": "User not found"
}
```

**Data:**

```json
{
    "models/user.py": {
        "id": "integer",
        "username": "string(80)",
        "email": "string(120)"
    },
    "services/user_service.py": {
        "__init__.py": "",
        "create_user": "user_data: dict -> User object",
        "get_user_by_id": "user_id: int -> User object"
    },
    "routes/user_views.py": {
        "__init__.py": "",
        "/users/<int:user_id> GET": "user_id: integer -> JSON object with user details or error message"
    }
}
```

**Flow:**

1. Yes
2. No
3. User GET /users/1
4. Is User Found?
5. Return User Data
6. Return Error 404
7. End
8. End

