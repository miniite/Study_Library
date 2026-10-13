---
type: "practice"
difficulty: "Basic"
---

# How would you build a Flask /chat endpoint that forwards a user message to an LLM API like OpenAI or Anthropic and returns the model's reply?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Streaming Responses](<../../../../TOOLS/Python/Flask/Streaming_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a Flask chat endpoint that forwards a user message to an LLM API like OpenAI or Anthropic, you need to follow several steps from setting up your Flask application to handling HTTP requests and integrating with external APIs. The key idea is to create a simple web service using Flask that acts as a frontend proxy to forward the request to the LLM API and then return the response.

#### Triage
The question is asking how to implement an endpoint in a Flask application that takes a user message, forwards it to an LLM API (such as OpenAI's ChatGPT or Anthropic's Claude), processes the response, and returns it back to the user. This involves understanding HTTP requests, setting up a Flask app, making HTTP calls using libraries like requests, handling JSON data, and managing potential errors.

#### Assumptions
We assume that you have basic knowledge of Python and are familiar with the Flask framework. The LLM API requires an API key or token for authentication. We will focus on using OpenAI’s API as a concrete example.

Step-by-Step Solution:

Install Dependencies: Ensure you have Flask installed in your environment.

BASH
pip install flask requests

Setup the Flask Application:

Define the endpoint for receiving user messages.
Integrate with OpenAI’s API to send the message and receive the response.

Create a Simple Flask App: Here's an example of how you might structure your code:

PYTHON
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

# Replace with your own API key from OpenAI
OPEN_AI_API_KEY = "YOUR_OPEN_AI_API_KEY"

@app.route('/chat', methods=['POST'])
def chat():
    # Extract the user message from the POST request
    user_message = request.json.get('message')

    if not user_message:
        return jsonify({"error": "Message is required"}), 400

    # Define the API endpoint for OpenAI ChatGPT
    openai_url = 'https://api.openai.com/v1/engines/davinci-codex/completions'

    # Prepare headers with your API key and content type
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {OPEN_AI_API_KEY}"
    }

    # Define the payload to send the user message to OpenAI API
    data = {
        "prompt": user_message,
        "max_tokens": 50
    }

    # Make a POST request to the OpenAI API
    response = requests.post(openai_url, headers=headers, json=data)

    if response.status_code == 200:
        # Return the generated text from the API response
        return jsonify({"response": response.json().get("choices", [{}])[0].get("text")})
    else:
        return jsonify({"error": f"Error: {response.status_code}"}), response.status_code

if __name__ == '__main__':
    app.run(debug=True)
Testing the Endpoint: You can test your endpoint using tools like curl, Postman, or by making HTTP POST requests with a JSON payload.
BASH
curl -X POST http://localhost:5000/chat \
     -H "Content-Type: application/json" \
     -d '{"message": "Hello, how are you?"}'

#### Closing Summary
and Common Mistakes:

Ensure that the API key is properly managed and not exposed in your code or version control.
Handle edge cases such as missing or malformed user messages.
Validate responses from the LLM API to ensure they are expected JSON structures.

