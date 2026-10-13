---
type: "practice"
difficulty: "Basic"
---

# How would you make an HTTP health-check request to a service URL in Python and decide if it is up based on the status code?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parsing JSON Safely](<../../../../TOOLS/Python/Python_Language/Parsing_JSON_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To perform an HTTP health check using Python, we need to send a GET request to the specified service URL and evaluate its response status code. A typical successful HTTP request returns a status code in the range 200-299, which indicates that the resource is available.

#### Triage
The question is asking how to implement an HTTP health check using Python. The key elements are:

Sending an HTTP GET request.
Handling the response from the server.
Checking the status code to determine if the service is up.

#### Assumptions

We will use a simple and reliable HTTP client library, such as requests.
The URL provided is valid and accessible.
A 200-299 range indicates that the service is healthy; any other response indicates it may be down.

#### First-principles solution

Import the necessary libraries: requests for making HTTP requests.
Define a function to perform the health check.
Send an HTTP GET request to the specified URL.
Check the status code of the response.
Return a boolean indicating whether the service is up (based on the status code).

#### Closing summary
and common mistakes: This solution ensures that we can quickly determine if a service is operational by inspecting its HTTP response. Common mistakes include not handling different status codes properly or forgetting to import necessary libraries.

**Code:**

```python
import requests

def check_service_health(url):
    try:
        # Send an HTTP GET request to the specified URL
        response = requests.get(url)

        # Check if the service is up based on the status code
        if 200 <= response.status_code < 300:
            return True, f"Service is up with status code: {response.status_code}"
        else:
            return False, f"Service might be down. Status code: {response.status_code}"
    except requests.RequestException as e:
        # Handle exceptions that may occur during the request
        return False, f"Error occurred: {e}"
```

**Example input / output:**

```
Example usage and dry-run

url = "http://example.com/health" result, message = check_service_health(url) print(f"Result: {result}, Message: {message}")

CODE
Copy

Output:

Result: True, Message: Service is up with status code: 200
```

