---
type: "practice"
difficulty: "Basic"
---

# How would you build a Flask classification endpoint that uses an LLM to label incoming support tickets by category?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Streaming Responses](<../../../../TOOLS/Python/Flask/Streaming_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a Flask classification endpoint using a Large Language Model (LLM) to label incoming support tickets, we need to follow these steps:

#### Overview
We'll create a basic Flask web application that takes an input ticket as JSON, passes it through the LLM for classification, and returns the predicted category.

#### Triage
The question is asking us to set up a simple API endpoint in Flask that interacts with an LLM. This involves understanding how Flask handles HTTP requests, integrating with an LLM like Anthropic's Claude or Anthropic's claude-api, and handling JSON inputs/output.

#### First-Principles Solution
Set up the Flask application structure.
Define a route for receiving tickets as POST requests.
Integrate the LLM to receive input, process it, and return the predicted category.

#### Closing Summary
+ Common Mistakes/Follow-Ups: Ensure proper error handling, validate inputs, and consider scalability.

Let's break this down step-by-step:

Setting Up Flask

First, we need to install Flask:

SH
pip install flask

Next, set up a basic Flask application structure in app.py:

PYTHON
from flask import Flask, request, jsonify
import requests  # Assuming LLM API is external
import json

app = Flask(__name__)

@app.route('/ticket_classification', methods=['POST'])
def ticket_classification():
    try:
        data = request.get_json()
        ticket_content = data['content']

        # Call the LLM with the ticket content to get a predicted category
        prediction = call_llm(ticket_content)

        return jsonify({"category": prediction})

    except Exception as e:
        return str(e), 400

def call_llm(content):
    # Example API endpoint for Anthropic's Claude (replace with actual LLM API)
    response = requests.post(
        "https://api.anthropic.com/v1/complete",
        headers={"Authorization": f"Bearer YOUR_API_KEY"},
        json={"prompt": content, "model": "claude-2"}
    )
    result = response.json()
    return result['completion'].strip().lower()  # Assuming completion contains the category

if __name__ == '__main__':
    app.run(debug=True)
Explanation of Each Part:
Import Libraries: We import Flask, requests, and json for JSON handling.
Flask Setup: Initialize the Flask application and define a route /ticket_classification.
Request Handling: Extract the ticket content from the incoming JSON payload.
LLM Integration: Use a function to call the LLM API with the ticket content, which returns a predicted category.
Error Handling: Catch any exceptions and return an error message.

#### Common Mistakes/Follow-Ups
Error Handling: Ensure robust error handling for invalid inputs or API failures.
Security: Secure your API by adding authentication mechanisms.
Performance: Consider caching or rate limiting if the LLM requests are frequent.
Scalability: Use a cloud service like AWS, GCP, or Azure to host the Flask application.

