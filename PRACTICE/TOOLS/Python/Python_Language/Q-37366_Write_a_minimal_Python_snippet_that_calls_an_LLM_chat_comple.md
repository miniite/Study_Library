---
type: "practice"
difficulty: "Basic"
---

# Write a minimal Python snippet that calls an LLM chat completion API and prints the model's text reply. What are the essential parameters you pass?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming LLM Responses](<../../../../TOOLS/Python/Python_Language/Streaming_LLM_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with language models, one of the most common tasks is calling an API to get a response based on some input prompt. In this case, we need to write a minimal Python snippet that calls an LLM (Language Model) chat completion API and prints out the model's text reply. The essential parameters you would typically pass include the API endpoint URL, the input message (prompt), and possibly your authentication credentials.

Let's break down the process step by step:

#### Triaging the Question

The question is essentially asking us to write a simple Python script that interfaces with an LLM chat completion API. The key aspect being probed here is understanding how APIs work in general, particularly RESTful or GraphQL-based APIs. We need to understand the parameters required for such calls and how to handle responses.

Essential Parameters
API Endpoint URL: This is the base URL where your model's endpoint resides.
Prompt: The input message that you want the LLM to generate a response for.
Authorization Token: Often, APIs require authentication tokens or API keys to ensure secure access.
Model Name (Optional): Sometimes, the API allows specifying which model variant to use.

#### First-Principles Solution

Here's a basic Python snippet using requests to call an LLM chat completion API:

PYTHON
import requests

# Define the API endpoint URL and authentication token
api_url = "https://example-api.com/chat"
auth_token = "your_api_key_here"

# Input prompt for the model
prompt = "Hello, how are you?"

# Prepare headers with authorization token if required
headers = {
    'Authorization': f'Bearer {auth_token}',
    'Content-Type': 'application/json'
}

# Define the payload (data) to be sent in the request body
payload = {
    "message": prompt
}

# Make a POST request to the API endpoint
response = requests.post(api_url, headers=headers, json=payload)

# Check if the request was successful
if response.status_code == 200:
    # Print the model's text reply
    print(response.json()["text"])
else:
    print(f"Request failed with status code {response.status_code}")

#### Closing Summary
+ Common Mistakes/Follow-Ups

This snippet demonstrates a simple way to call an LLM chat completion API using Python. The key parameters here are the API endpoint URL, the prompt message, and authentication if required.

#### Common mistakes
include:

Forgetting to set the Content-Type header.
Not handling errors properly (e.g., checking response status codes).
Not including necessary headers like authorization tokens.

For follow-ups, one might explore more complex scenarios such as rate limiting, handling different HTTP methods, or dealing with larger input data. Additionally, integrating error handling and logging can be crucial for production-level code.

