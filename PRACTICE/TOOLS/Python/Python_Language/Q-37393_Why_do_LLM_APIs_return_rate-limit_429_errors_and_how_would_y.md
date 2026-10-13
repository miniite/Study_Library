---
type: "practice"
difficulty: "Intermediate"
---

# Why do LLM APIs return rate-limit (429) errors, and how would you add a simple retry with a short wait in your Python client?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Rate-limited Integrations](<../../../../TOOLS/Python/Python_Language/Rate-limited_Integrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rate-limit errors (HTTP 429) occur when an API receives too many requests from a single source within a specified time frame, indicating that the request rate exceeds what is allowed by the service provider's policies. This can be due to various reasons such as unexpected spikes in traffic or violating usage quotas.

To handle this scenario effectively in a Python client, we need to implement a simple retry mechanism with exponential backoff. Here’s how we would approach this problem step-by-step:

Understand the Rate-Limit Error: The 429 status code indicates that the server is refusing to process a request because it has been sent by an API client that has exceeded its rate limit.
Implement Retry Logic: We will add retry logic in our Python client, allowing the application to wait for a short period before re-attempting the request if we receive a 429 error.

#### Triage

The question is essentially asking how to manage and recover from API rate-limit errors by implementing a retry mechanism with exponential backoff. The interviewer is probing your understanding of HTTP status codes, error handling, and basic client-server interaction in Python. Key assumptions include:

We have access to the API response.
The rate limit information (if available) can be used to determine when it’s safe to retry.

#### First-Principles Solution
Check for 429 Error: When making an API request, check if the HTTP response status code is 429.
Handle 429 Response: If a 429 error occurs, extract any relevant information from the Retry-After header or JSON body to determine how long we should wait before retrying.
Implement Exponential Backoff: Use exponential backoff to avoid overwhelming the server with repeated requests too quickly.
Python Code Example

Here’s a basic implementation using the requests library in Python:

PYTHON
import time
from requests import RequestException

def make_request(url, headers):
    for attempt in range(3):  # Maximum of 3 retries
        try:
            response = requests.get(url, headers=headers)
            if response.status_code == 429:
                retry_after = int(response.headers.get('Retry-After', '5'))  # Default to 5 seconds
                print(f"Rate limit hit. Retrying in {retry_after} seconds...")
                time.sleep(retry_after)
            else:
                return response.json()  # Process the successful response
        except RequestException as e:
            print(f"Request failed: {e}")
    raise Exception("Failed to make request after multiple retries")

# Example usage
url = "https://api.example.com/data"
headers = {"Authorization": "Bearer <your_token>"}
response_data = make_request(url, headers)
print(response_data)

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By implementing retry logic with exponential backoff in our Python client, we can manage rate-limit errors effectively, ensuring that the application does not overwhelm the API server.

#### Common Mistakes
Forgetting to handle non-rate-limited errors.
Not setting a maximum number of retries, leading to infinite loops.
Ignoring the Retry-After header and always waiting for a fixed period.

#### Follow-Ups

The interviewer might ask about handling other HTTP status codes or more advanced techniques like adaptive backoff based on historical request success rates. They could also probe further into scaling considerations if you are at an Architect/Principal level.

