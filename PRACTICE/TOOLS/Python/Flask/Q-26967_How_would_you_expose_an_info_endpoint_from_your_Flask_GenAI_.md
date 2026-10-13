---
type: "practice"
difficulty: "Basic"
---

# How would you expose an /info endpoint from your Flask GenAI service that reports the active model, provider, and embedding dimension?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Testing Flask Endpoints](<../../../../TOOLS/Python/Flask/Testing_Flask_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To expose an /info endpoint in a Flask GenAI service, we need to create a route that returns information about the current state of the model being used. This includes details such as the active model name, provider (e.g., Hugging Face), and embedding dimension. Let's walk through the steps required from first principles.

#### Triaging the Question

The question is asking how to set up an endpoint in a Flask application that returns specific metadata about the service. Specifically, it wants us to return information such as the active model name, provider, and embedding dimensions. The interviewer is probing your understanding of basic web development with Flask, particularly setting up routes and returning JSON data.

Step-by-Step Solution
Set Up a Basic Flask Application: Ensure you have a running Flask application.
Define an Endpoint: Create a route for the /info endpoint that returns the required information.
Return Data as JSON: Use Flask's jsonify to return data in a structured format.
Implementing the Solution

Let’s assume we already have a basic Flask setup with a model and embedding dimension defined:

PYTHON
from flask import Flask, jsonify

app = Flask(__name__)

# Simulated state of the service
model_name = "gpt-3.5-turbo"
provider = "Anthropic"
embedding_dimension = 768

@app.route('/info')
def info():
    """Endpoint to return information about the active model and provider."""
    return jsonify({
        'model': model_name,
        'provider': provider,
        'embedding_dimension': embedding_dimension
    })

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
+ Common Mistakes/Follow-Ups

The /info endpoint returns a JSON object with the active model name, provider, and embedding dimension. This is useful for clients to know which service they are interacting with and what capabilities it has.

#### Common mistakes
include:

Forgetting to import jsonify.
Not properly defining the route and function.
Returning plain text instead of JSON.

#### Follow-up
questions might include how you would update this information in real-time or handle different models dynamically.

