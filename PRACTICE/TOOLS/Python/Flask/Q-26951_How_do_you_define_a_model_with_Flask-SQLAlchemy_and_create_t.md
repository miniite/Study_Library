---
type: "practice"
difficulty: "Intermediate"
---

# How do you define a model with Flask-SQLAlchemy and create the tables?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Flask-SQLAlchemy](<../../../../TOOLS/Python/Flask/Flask-SQLAlchemy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define a model with Flask-SQLAlchemy and create the corresponding database tables, you need to follow several steps. First, you'll set up your Flask application and integrate SQLAlchemy as the ORM (Object-Relational Mapping) tool. Then, you'll define your models using SQLAlchemy's declarative base. Finally, you can run a migration command to create the tables in your database.

To begin, we assume that you have a basic understanding of setting up a Flask application and working with Python. The key idea here is to use SQLAlchemy's ORM capabilities to map your Python classes (models) directly into database tables.

First, let's triage what this question is asking:

What the question probes: Understanding how to integrate SQLAlchemy in a Flask project.

#### Assumptions
You have an existing Flask application and a basic understanding of databases and models.
Steps: Define your model classes using SQLAlchemy's declarative syntax, configure the database URI, and run migrations.

Now, let's walk through the process step-by-step:

Setup Your Flask Application: Ensure you have Flask installed in your environment. If not, install it via pip:

BASH
pip install Flask

Install SQLAlchemy and Flask-SQLAlchemy: Add Flask-SQLAlchemy to your project dependencies:

BASH
pip install Flask-SQLAlchemy

Define Your Models: Create a file (e.g., models.py) where you will define your models using the declarative base provided by SQLAlchemy.

Configure the Database URI: In your Flask application configuration, set up the database connection string.

Create and Initialize the Database Tables: Run migration commands to create and initialize the tables in your database.

Here is a detailed example:

Step 1: Setup Your Flask Application

Create a file named app.py for your main application:

PYTHON
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
# Configure the database URI (replace with your actual DB URI)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

if __name__ == '__main__':
    app.run(debug=True)
Step 2: Define Your Models

In a file named models.py, define your models:

PYTHON
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    user = db.relationship('User', backref=db.backref('posts', lazy=True))
Step 3: Configure the Database URI

In app.py, ensure you have configured the database URI:

PYTHON
from app import db

# Create the database and tables
with app.app_context():
    db.create_all()
Step 4: Running Migrations

To create the tables, run the following command in your terminal:

BASH
flask db init
flask db migrate -m "Initial migration."
flask db upgrade

These commands will initialize a database migration environment and generate an initial migration. Then, you apply that migration to update the database schema.

#### Closing Summary

In summary, defining models with Flask-SQLAlchemy involves setting up your Flask application, configuring the database URI, creating model classes, and running migrations to create tables in the database. By following these steps, you can effectively manage database operations within a Flask application using SQLAlchemy as the ORM.

#### Common mistakes
include forgetting to configure the database URI correctly or not running the migration commands properly. Always ensure that your development environment matches your production setup for consistency.

**Code:**

```

from flask import Flask from flask_sqlalchemy import SQLAlchemy

app = Flask(name)

Configure the database URI (replace with your actual DB URI)

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db' app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

if name == 'main': app.run(debug=True)

CODE
Copy

```python
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    user = db.relationship('User', backref=db.backref('posts', lazy=True))
```

**Data:**

```yaml
# Example database URI configuration
sqlalchemy_database_uri: 'sqlite:///example.db'
```

