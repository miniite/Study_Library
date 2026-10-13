---
type: "practice"
difficulty: "Intermediate"
---

# A gateway's rate-limit response returns 200 with an error body due to a misconfig, so clients keep retrying as if success; how do you audit that gateway responses use correct status codes and catch this class of bug in CI?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Response Transformation and Upstream Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a backend system using API Gateway (APIG), ensuring accurate status code responses is crucial, especially when rate-limiting or other misconfigurations occur. In this scenario, we need to ensure that the gateway returns correct HTTP status codes, particularly 4xx and 5xx errors for failures like rate limits, so clients do not mistakenly treat them as successes. We will focus on auditing this behavior in CI (Continuous Integration).

#### Triaging the Question

The core of the question is about ensuring accurate HTTP status code responses from an API Gateway during CI to catch misconfigurations that lead to incorrect response handling by clients. This involves:

Understanding Status Codes: Ensuring the gateway uses the correct 4xx or 5xx codes for failure conditions.
Misconfiguration Detection: Identifying and fixing configurations where the gateway incorrectly returns a 200 OK status despite an error condition.
CI Integration: Setting up CI to automatically test these scenarios, ensuring reliable deployment.

#### First-Principles Solution

Define Correct Behavior:

The API Gateway should return appropriate HTTP status codes based on the failure type.
Misconfigurations such as rate limits should result in a 429 Too Many Requests or possibly a 503 Service Unavailable, not 200 OK.

Implement Status Code Assertions:

Use CI tools to automatically test gateway responses and verify status codes match expected values.
For example, when simulating rate limit failures, the CI should check that the response includes a 429 or 503 status code.

Automated Testing:

Create automated tests in your CI pipeline that send requests to the gateway under various failure conditions (e.g., simulated rate limits) and verify the returned status codes.
Use tools like curl, Postman, or custom scripts to simulate API calls and capture response headers.

CI Integration Steps:

In your CI setup, add steps that trigger these automated tests.
Integrate with a tool like JUnit for Java, PyTest for Python, or any other framework you use, which can run these tests as part of the build process.
Log results and alert if status codes are incorrect.
Detailed Implementation

To implement this solution, follow these steps:

Simulate Rate Limit Failures:

BASH
# Example using curl with a custom header to simulate rate limit failure
curl -H "X-Rate-Limit-Exceeded: true" https://api-gateway.example.com/endpoint -v

Verify Response Status Codes: Ensure the CI tool captures and logs the response status codes.

BASH
# Example using a bash script to verify status code
STATUS_CODE=$(curl -s -w "%{http_code}" -o /dev/null https://api-gateway.example.com/endpoint)
echo "Status Code: $STATUS_CODE"
if [ "$STATUS_CODE" != "429" ]; then
  echo "Failed: Expected 429, got $STATUS_CODE"
  exit 1
fi

Integrate with CI Pipeline: Add the above steps as part of your CI pipeline.

YAML
- name: Test API Gateway Response Codes
  run: |
    STATUS_CODE=$(curl -s -w "%{http_code}" -o /dev/null https://api-gateway.example.com/endpoint)
    echo "Status Code: $STATUS_CODE"
    if [ "$STATUS_CODE" != "429" ]; then
      exit 1
    fi

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that any misconfiguration in the API Gateway, such as incorrectly returning a 200 OK status for rate limit errors, is caught during CI. Key common mistakes include:

Failing to simulate all relevant failure conditions.
Not integrating status code verification into the CI pipeline.
Misconfiguring the gateway's response settings.

For follow-ups, consider extending this approach to other types of failures and adding more comprehensive logging and alerts.

**Comparison:**

```
Test Case	Expected Status Code
Rate Limit Exceeded	429
Service Unavailable	503
```

**Code:**

```bash
# Example Bash Script for Testing Status Codes
STATUS_CODE=$(curl -s -w "%{http_code}" -o /dev/null https://api-gateway.example.com/endpoint)
echo "Status Code: $STATUS_CODE"
if [ "$STATUS_CODE" != "429" ]; then
  echo "Failed: Expected 429, got $STATUS_CODE"
  exit 1
fi
```

**Example input / output:**

```text
Input:
curl -H "X-Rate-Limit-Exceeded: true" https://api-gateway.example.com/endpoint

Output (Expected):
Status Code: 429
```

**Data:**

```json
{
  "url": "https://api-gateway.example.com/endpoint",
  "header": {
    "X-Rate-Limit-Exceeded": "true"
  }
}
```

**Flow:**

1. Yes
2. No
3. Simulate API Call
4. Is the response status code 4xxor 5xx?
5. Log and Alert
6. Failed Test, Exit with Error
7. Passes Test

