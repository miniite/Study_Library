---
type: "practice"
difficulty: "Basic"
---

# How would you write a smoke test that hits your deployed FastAPI LLM endpoint after a deploy to confirm it responds?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Batching LLM Requests](<../../../../TOOLS/Python/FastAPI/Batching_LLM_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a smoke test for your FastAPI Language Model (LLM) endpoint, we need to ensure that our deployed model is functioning correctly right after deployment. A basic smoke test will send an HTTP request to the endpoint and verify if it returns a response within expected time limits. Here’s how you can approach this task from first principles:

Understand What We’re Testing: The primary goal is to confirm that the FastAPI server hosting our LLM model is up and running, and that it can process requests.

Identify Key Assumptions:

The endpoint URL is known.
The model returns a valid JSON response with expected data fields.
We have access to tools like curl or Python’s requests library for making HTTP requests.

#### First-Principles Solution

Define the Endpoint: Identify the URL of your deployed FastAPI server where the LLM model is hosted.
Send a Test Request: Use a tool like curl to send a simple GET request to this endpoint.
Check Response Time: Ensure that the response time is within acceptable limits (e.g., less than 5 seconds).
Validate Response Content: Verify that the response contains valid JSON with expected fields.

Detailed Steps:

Install requests if you prefer using Python over curl.
Write a simple script to send the request and handle the response.
Use assertions to check the status code, content type, and presence of key fields in the JSON.

Here’s an example Python script using the requests library:

PYTHON
import requests

# Define the endpoint URL
endpoint_url = "http://your-fastapi-app.com/api/llm-endpoint"

def send_smoke_test_request(url):
    # Send a GET request to the specified URL
    response = requests.get(url)

    # Check if the response status code is 200 OK
    assert response.status_code == 200, f"Expected status code 200, got {response.status_code}"

    # Ensure the content type is JSON
    assert 'application/json' in response.headers['Content-Type'], "Response should have a content-type of application/json"

    # Parse and validate the JSON response
    json_response = response.json()
    expected_keys = ["status", "output"]
    for key in expected_keys:
        assert key in json_response, f"Expected key '{key}' not found in JSON response"

# Send the smoke test request
send_smoke_test_request(endpoint_url)

#### Closing Summary
and Common Mistakes:
The main goal of this script is to verify that your FastAPI LLM endpoint responds correctly.

#### Common mistakes
include forgetting to check the status code, not validating the content type, or overlooking key fields in the JSON response.

