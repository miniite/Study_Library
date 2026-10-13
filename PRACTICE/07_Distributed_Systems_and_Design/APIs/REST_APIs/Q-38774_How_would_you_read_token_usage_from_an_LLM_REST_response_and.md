---
type: "practice"
difficulty: "Intermediate"
---

# How would you read token usage from an LLM REST response and log it so you can track how much each request costs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming API Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Streaming_API_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To read token usage from an LLM REST response and log it, we need to parse the response data, extract the relevant token count information, and then store or log this data. This process involves understanding the structure of the API response and implementing a logging mechanism.

#### Triaging the Question

The core of the question is about extracting specific data (token usage) from an LLM REST API response and storing it for future reference. The interviewer may be probing your understanding of:

API Responses: How to parse JSON responses.
Logging Mechanisms: What tools or methods are available for logging information.
Error Handling: Ensuring the system can handle different response scenarios gracefully.

#### First-Principles Solution

Understand API Response Structure:

Assume the LLM REST API returns a JSON object with fields like response, token_usage, and request_id.

Parse the JSON Response:

Use Python's built-in json module to parse the response.

Extract Token Usage Data:

Access the token_usage field in the parsed JSON.

Log the Data:

Store or log this data using a logging framework like logging in Python.

#### Example
Code
PYTHON
import json
import requests
from datetime import datetime

# Function to fetch and process token usage from API response
def read_and_log_token_usage(api_url, api_key):
    headers = {"Authorization": f"Bearer {api_key}"}

    try:
        response = requests.get(api_url, headers=headers)

        if response.status_code == 200:
            json_data = response.json()

            # Extract token usage from the JSON data
            token_usage = json_data['token_usage']

            # Log the token usage and request ID
            log_token_usage(token_usage, json_data['request_id'])
        else:
            print(f"Failed to fetch data: {response.status_code}")
    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")

# Function to log token usage
def log_token_usage(token_usage, request_id):
    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')

    # Example logging format
    log_message = f"{timestamp}, Request ID: {request_id}, Token Usage: {token_usage}"

    with open('token_usage.log', 'a') as log_file:
        log_file.write(log_message + '\n')

# Example usage
api_url = "https://example-llm-api.com/endpoint"
api_key = "your_api_key_here"
read_and_log_token_usage(api_url, api_key)

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution covers the basics of reading token usage from an LLM REST response and logging it. However, there are several areas to consider:

Error Handling: Ensure robust error handling for network issues or unexpected API responses.
Logging Mechanism: Consider using a more sophisticated logging system like structlog or setting up centralized logging in a cloud environment.
Scalability: For high-scale applications, you might want to store token usage data in a database instead of writing it to a file.

#### Common Mistakes
Ignoring Error Handling: Not handling API response codes and network errors can lead to unexpected failures.
Inadequate Logging: Using plain text files for logging may not be suitable for large-scale applications, especially if you need to integrate with other systems or monitoring tools.
Insufficient Data Storage: Writing token usage directly to a file might be problematic in terms of scalability and maintainability.
Closing

Always ensure your solution accounts for potential edge cases and scales appropriately for the expected load.

