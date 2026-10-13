---
type: "practice"
difficulty: "Intermediate"
---

# How would you rate-limit a public Flask LLM endpoint so a single client cannot run up your token bill?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Streaming Responses](<../../../../TOOLS/Python/Flask/Streaming_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To rate-limit a public Flask LLM (Language Model) endpoint, we need to ensure that a single client does not abuse the system and rack up excessive token costs. This involves identifying the client, tracking requests, and enforcing limits based on this information. Here’s how you can implement this:

#### Triaging the Question

The core of the question is about setting rate limits for an API endpoint in Flask to prevent abuse. The interviewer might be looking at your understanding of HTTP headers, IP addressing, authentication mechanisms (like API keys), and possibly a caching layer or rate-limiting library.

#### Assumptions
We are dealing with public endpoints that do not require user authentication.
The primary identifier for a client is its IP address.
We want to limit the number of requests per minute/IP.

#### First-Principles Solution

To achieve this, we can use Flask middleware or an external rate-limiting library like Flask-Limiter. Here’s a step-by-step approach:

Install Flask and Flask-Limiter: Ensure you have Flask and Flask-Limiter installed in your environment.
Configure Flask Application: Set up the Flask app with necessary configurations.
Implement Rate Limiting: Use @limiter.limit decorator to apply rate limits to specific routes.

#### Implementation
Step 1: Install Dependencies
BASH
pip install flask flask-limiter
Step 2: Configure Flask Application

Here’s a basic setup:

PYTHON
from flask import Flask, request
from flask_limiter import Limiter
import os

app = Flask(__name__)

# Set up rate limiters
limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

def get_remote_address():
    return request.remote_addr

@app.route('/api/v1/llm', methods=['POST'])
@limiter.limit("5 per minute")  # Limit to 5 requests per minute
def llm_endpoint():
    # Your LLM logic here
    response = {"message": "Request processed"}
    return response, 200

if __name__ == "__main__":
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups

This implementation sets a basic rate limit of 5 requests per minute for the /api/v1/llm endpoint. However, there are several common mistakes to avoid:

Over-reliance on IP Address: This approach may not work well in environments with shared IPs or cloud services.
Ignoring User-Agent Headers: In some cases, you might want to consider combining IP address and user-agent for more accurate rate limiting.
Rate Limiting at the Application Level: Sometimes it’s better to handle rate limits at a higher level (e.g., API gateway) before they hit your application.

#### Follow-Ups
How would you handle rate limiting if the client is using an API key instead of IP address?
What are some other strategies for dealing with abusive behavior in your Flask app?

