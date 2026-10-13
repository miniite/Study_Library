---
type: "practice"
difficulty: "Intermediate"
---

# How do you use url_for() to reference an endpoint that lives inside a blueprint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Routing and Blueprints](<../../../../TOOLS/Python/Flask/Routing_and_Blueprints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, let's start with a brief overview: url_for() in Flask is used to generate URLs for endpoints based on their name and arguments. When working with blueprints, we need to ensure that we correctly reference the endpoint within its blueprint context. Here’s how you can do it.

First, I'll break down what the question is asking and highlight key points:

Understanding url_for(): This function helps in generating URLs for endpoints.
Blueprints Context: Blueprints are a way to organize your Flask application into smaller modules. Each blueprint has its own namespace, which affects how you reference endpoints within it.

Now, let's reason from first principles and derive the full solution:

Step-by-Step Solution

Define a Blueprint:

PYTHON
from flask import Blueprint

# Create a blueprint named `blog`
blog_bp = Blueprint('blog', __name__)

Register an Endpoint within the Blueprint:

PYTHON
@blog_bp.route('/post/<int:id>')
def post(id):
    return f"Post with ID {id}"

Using url_for() to Reference the Endpoint:

Within a blueprint, you use the blueprint's name as the prefix.

#### Example
To get the URL for the endpoint within the same blueprint:

PYTHON
from flask import url_for

# In your route or template
@blog_bp.route('/recent')
def recent():
    return f'<a href="{url_for("blog.post", id=1)}">Link to Post 1</a>'

Outside the Blueprint Context:

If you need to reference an endpoint outside the blueprint, you can pass it directly by its name:

PYTHON
@app.route('/admin/post/<int:id>')
def admin_post(id):
    return f'<a href="{url_for("blog.post", id=id)}">View Blog Post</a>'

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, using url_for() with a blueprint involves prefixing the endpoint name with the blueprint’s name. Ensure you correctly reference endpoints within or outside blueprints.

#### Common mistakes
include:

Forgetting to prefix the endpoint name with the blueprint name.
Misunderstanding that url_for() is not just for URLs but also generates URLs based on the current URL context.

Additional follow-ups could involve discussing how to handle nested blueprints, static files, and different URL generation scenarios in complex applications.

