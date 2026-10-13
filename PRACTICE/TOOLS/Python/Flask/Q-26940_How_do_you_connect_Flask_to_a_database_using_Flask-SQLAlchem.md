---
type: "practice"
difficulty: "Intermediate"
---

# How do you connect Flask to a database using Flask-SQLAlchemy?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Flask-SQLAlchemy](<../../../../TOOLS/Python/Flask/Flask-SQLAlchemy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To connect Flask to a database using Flask-SQLAlchemy, we need to follow these steps:

#### Overview
This process involves setting up a basic Flask application and integrating it with SQLAlchemy via the Flask-SQLAlchemy extension.

#### Triage
The interviewer is probing your understanding of how to integrate ORM (Object-Relational Mapping) in a Flask application for database interaction. We will assume you have some basic knowledge of Flask and SQLAlchemy.
Step-by-step Solution:
Install Required Packages: First, ensure you have the necessary packages installed: Flask and Flask-SQLAlchemy.
Initialize Flask Application: Create a new Flask application instance.
Configure Database URI: Set up the connection to your database by configuring the SQLALCHEMY_DATABASE_URI.
Create Models: Define your data models using SQLAlchemy’s ORM capabilities.
Initialize Database with Migrations (Optional): Use Alembic for database migrations if necessary.

Here's a detailed breakdown:

Step 1: Install Required Packages
BASH
pip install Flask Flask-SQLAlchemy
Step 2: Initialize Flask Application

Create a file app.py and set up the basic structure:

PYTHON
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

# Initialize the Flask application
app = Flask(__name__)

# Configure database URI (e.g., SQLite, PostgreSQL)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db'

# Create a SQLAlchemy object instance and link it to our Flask app
db = SQLAlchemy(app)

# Define models using SQLAlchemy ORM
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

    def __repr__(self):
        return '<User %r>' % self.username

# Initialize the database
@app.before_first_request
def init_db():
    db.create_all()

if __name__ == '__main__':
    app.run(debug=True)
Step 3: Configure Database URI

In this example, we are using SQLite for simplicity. For a production application, you might use PostgreSQL or MySQL.

Step 4: Create Models

Define your data models by subclassing db.Model. The above example defines a simple User model with columns id, username, and email.

Step 5: Initialize Database with Migrations (Optional)

If your application will evolve, consider using Alembic for database migrations. First, install Alembic:

BASH
pip install alembic

Then set up the migration directory and run migrations.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Forgetting to configure SQLALCHEMY_DATABASE_URI correctly.

#### Follow-Up
Discuss potential issues like database connection pooling, transaction management, and handling connections in a production environment.

This setup provides a solid foundation for integrating SQLAlchemy with Flask. It's crucial to understand the basics of ORM, database interactions, and error handling in this context.

**Comparison:**

```
Column	Type	Constraints
id	Integer	Primary Key
username	String(80)	Unique, Not Null
email	String(120)	Unique, Not Null
Model Name	Description
User	Represents a user in the application.
```

**Code:**

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

# Initialize the Flask application
app = Flask(__name__)

# Configure database URI (e.g., SQLite, PostgreSQL)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db'

# Create a SQLAlchemy object instance and link it to our Flask app
db = SQLAlchemy(app)

# Define models using SQLAlchemy ORM
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

    def __repr__(self):
        return '<User %r>' % self.username

# Initialize the database
@app.before_first_request
def init_db():
    db.create_all()

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```

To check if the setup works, you can run the application and use a tool like SQLite Browser to inspect the database.

Run the Flask app:

CODE
Copy
python app.py

Check the SQLite database file example.db (it should be in the same directory as your script).

Use SQLite commands or tools to verify that the tables have been created.
```

**Data:**

```json
{
  "SQLALCHEMY_DATABASE_URI": "sqlite:///example.db",
  "models": [
    {
      "name": "User",
      "columns": [
        {"name": "id", "type": "Integer", "primary_key": true},
        {"name": "username", "type": "String(80)", "unique": true, "nullable": false},
        {"name": "email", "type": "String(120)", "unique": true, "nullable": false}
      ]
    }
  ]
}
```

**Flow:**

1. Start Flask Application
2. ConfigureSQLALCHEMY_DATABASE_URI
3. Create SQLAlchemy instanceand link to Flask app
4. Define models usingSQLAlchemy ORM
5. Initialize database with'db.create_all()' at startup
6. Run application

