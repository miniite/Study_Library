---
type: "practice"
difficulty: "Intermediate"
---

# How would you set a timeout on your REST call to an LLM API, and why is that important when a model is slow to respond?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling 429 Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Handling_429_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set a timeout on a REST call to an LLM (Language Model) API, you would configure the client making the request to include a timeout parameter. This ensures that if the response from the model does not arrive within the specified time limit, the request is automatically canceled, and an appropriate error or fallback action is triggered.

#### Triage

The question asks how to set up a timeout mechanism for a REST API call to an LLM. The key aspects include understanding why timeouts are necessary when dealing with slow responses from models. We will discuss common scenarios where this issue arises and walk through the implementation steps.

#### Assumptions
You have access to the codebase or configuration of the client making the API request.
The LLM API uses a synchronous HTTP method (like GET, POST) for its communication.

#### First-Principles Solution

When making an HTTP call using libraries such as requests in Python, you can set a timeout parameter. Here’s how you would do it:

Identify the Client Library:
If you are using requests, which is a common HTTP library in Python.
Set the Timeout Parameter:
When making the API call, include a timeout argument.
PYTHON
import requests

def make_api_call(url):
    try:
        response = requests.get(url, timeout=5)  # Set timeout to 5 seconds
        return response.json()  # Process the JSON response if applicable
    except requests.Timeout:
        print("The request timed out")
    except Exception as e:
        print(f"An error occurred: {e}")

url = "https://api.example.com/llm"
make_api_call(url)

#### Closing Summary
and Common Mistakes/Follow-Ups

Setting a timeout is crucial when interacting with external APIs, especially LLMs that might be slow or unresponsive. This prevents the client application from hanging indefinitely on a slow response, allowing for proper error handling and graceful degradation of the system.

#### Common Mistakes
Not Setting Any Timeout: Leaving no timeout can lead to long-running requests, blocking other tasks.
Setting an Unreasonable Timeout: Choosing too short a timeout might result in frequent false timeouts due to network latency or minor processing delays.
Ignoring Exception Handling: Not catching and handling exceptions properly can make your application more prone to crashes.

#### Follow-Ups
What happens if the LLM model is consistently slow?

#### Cons
ider implementing retries with exponential backoff to handle persistent slowness without overwhelming the system.
How do you ensure that timeouts are configurable for different environments?
Use environment variables or configuration files to make timeout values adjustable based on deployment context.

