---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python health-check script that returns exit code 0 if a service is healthy and 1 otherwise, for use as a container healthcheck?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Graceful Shutdown](<../../../../TOOLS/Python/Python_Language/Graceful_Shutdown.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a Python health-check script for a containerized application, we need to ensure it can be run from within the container and return an appropriate exit code based on the service's health status. This script will perform a simple check (such as verifying if a specific endpoint is reachable) and exit with 0 if healthy or 1 otherwise.

#### Triage

The question asks for a Python script that serves as a healthcheck, returning exit code 0 for healthy services and exit code 1 for unhealthy ones. The script will be run inside the container to monitor its health status.

#### First-Principles Solution

We'll write a simple Python script using the requests library to perform an HTTP GET request to a predefined endpoint within our service. If the response is successful (HTTP 200), we return exit code 0; otherwise, we return exit code 1.

Step-by-Step Solution
Install Required Libraries: Ensure Python and requests are available in your container.
Define Endpoint URL: Specify a reliable endpoint within the service that is expected to be always up when the service is healthy.
Perform HTTP Request: Use requests.get() to send an HTTP GET request to the defined URL.
Check Response Status Code: If the status code is 200, return exit code 0; otherwise, return exit code 1.
Python Script

Here's a simple implementation of the health-check script:

PYTHON
import requests

def check_health():
    # Define the service endpoint to be checked
    url = "http://service/api/health"

    try:
        response = requests.get(url)
        if response.status_code == 200:
            return 0  # Service is healthy
        else:
            return 1  # Service is unhealthy
    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")
        return 1  # Consider any exception as a failure

if __name__ == "__main__":
    exit_status = check_health()
    print(f"Health Check Exit Status: {exit_status}")

#### Closing Summary

The Python script performs an HTTP GET request to a predefined endpoint and checks the response status code. If it's 200, indicating successful communication with the service, the script returns exit code 0. Any other status or any exception results in exit code 1.

#### Common Mistakes/Follow-Ups
Missing Error Handling: Ensure all possible errors (e.g., network issues) are handled gracefully.
Inadequate Endpoint Selection: The chosen endpoint should always be up when the service is healthy, otherwise false negatives might occur.

#### Performance Considerations
For production systems, consider the performance impact of making HTTP requests within health checks.

