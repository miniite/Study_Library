---
type: "practice"
difficulty: "Intermediate"
---

# How would you use Flask-SQLAlchemy relationships to model a one-to-many between User and Order?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Flask-SQLAlchemy](<../../../../TOOLS/Python/Flask/Flask-SQLAlchemy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a one-to-many relationship between User and Order using Flask-SQLAlchemy, we'll need to define two models where each user can have multiple orders but each order belongs to exactly one user. This is a common scenario in e-commerce or subscription services. Let's break down the steps involved.

#### Triaging the Question

The core of this question is understanding how to set up relationships between tables using Flask-SQLAlchemy, which is an extension for SQLAlchemy that provides convenient facilities for common database operations on top of SQLAlchemy’s core features. The interviewer might be probing your knowledge of relational databases and ORM (Object-Relational Mapping) concepts.

#### Assumptions
We are working with a Flask application.
SQLAlchemy is the ORM we will use to interact with our database via Flask-SQLAlchemy.
User and Order are two models representing entities in our application.
A user can place multiple orders, but each order belongs to one user.

#### Solution

We'll start by defining the User model. This model needs a primary key id, which is unique for each user. Additionally, it will have an attribute or relationship that links back to many orders through orders.

PYTHON
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

    # One-to-Many relationship with Order
    orders = db.relationship('Order', backref='user', lazy=True)

Next, we define the Order model. This model requires a primary key and will have attributes like order_id, product_name, quantity, etc., along with a foreign key referencing the user who placed the order.

PYTHON
class Order(db.Model):
    __tablename__ = 'orders'

    id = db.Column(db.Integer, primary_key=True)
    product_name = db.Column(db.String(100), nullable=False)
    quantity = db.Column(db.Integer, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)

#### Closing Summary

By defining these relationships, we have established a one-to-many relationship between User and Order. Each User can have multiple Orders, but each Order is associated with only one User.

#### Common Mistakes/Follow-ups
Forgetting to Set the ForeignKey: Ensure that you set up the foreign key correctly in the Order model.
Not Using Lazy Loading: If you don't use lazy=True or another appropriate setting, your queries might be less efficient.
Assuming One-to-One vs Many-to-One: Make sure to understand the difference and choose the correct relationship type.

If asked for further details, common follow-ups could include:

How would you handle cascading deletes?
What if a user places an order, but then cancels it?
How do you query orders by specific users or products?

