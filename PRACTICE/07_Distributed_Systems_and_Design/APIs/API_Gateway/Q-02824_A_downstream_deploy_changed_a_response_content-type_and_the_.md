---
type: "practice"
difficulty: "Intermediate"
---

# A downstream deploy changed a response content-type and the gateway's compression plugin started double-encoding responses; how do you detect this at the edge and add a regression check so it can't ship again?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Response Transformation and Upstream Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a downstream deploy changed the response content-type, causing the API Gateway’s compression plugin to double-encode responses, we need to identify the root cause and add robust regression checks. At an intermediate level of experience, you should focus on understanding the interaction between the API Gateway and the backend service, and implementing automated tests to prevent future regressions.

Firstly, let's understand that the problem arises from a mismatch in content-type handling: the backend now returns a response with a different or unsupported content-type (e.g., application/json) while the gateway is configured to handle text/html. This can lead to double-encoding if not properly handled. To detect and prevent this, we'll need to:

Identify the Issue: Confirm that it's indeed a content-type mismatch causing the problem.
Implement Regression Tests: Add tests in the API Gateway configuration or backend service to ensure consistent handling of content-types.
Automate Testing: Integrate these checks into your CI/CD pipeline.

#### Triage
and Assumptions
The downstream deploy has changed a response's Content-Type header.
The API Gateway uses middleware for compression, which can introduce issues if not configured correctly.
The double-encoding issue is related to how the gateway handles content encoding based on the backend’s Content-Type.

#### First-Principles Solution

Identify the Mismatch:

Check the logs and response headers in the API Gateway to confirm that the Content-Type header has changed.
Ensure the gateway is configured correctly for different content types (e.g., application/json, text/html).

Implement Content-Type Handling Logic:

Modify or add middleware in your API Gateway configuration to handle different Content-Type headers appropriately.
For example, if you use AWS API Gateway, ensure the integration responses are correctly configured.

Automated Testing:

Write unit tests for the backend service to verify that it returns the correct content type and encoding.
Add gateway-level checks in your CI/CD pipeline to validate response headers during deployments.
Step-by-Step Solution

Check Logs and Headers:

PLAINTEXT
# Example log entry from API Gateway
[INFO] 2023-11-01T14:05:00Z - Content-Type: application/json; charset=utf-8

Confirm that the backend now returns a Content-Type of application/json, and ensure the gateway is configured to handle this.

Update Gateway Configuration:

JSON
{
  "mappings": [
    {
      "requestPath": "/api/v1/data",
      "integration": {
        "type": "HTTP_PROXY",
        "uri": "https://example.com/api/v1/data"
      },
      "responses": {
        "200": {
          "statusCode": 200,
          "responseParameters": {
            "method.response.header.Content-Type": true
          }
        }
      },
      "settings": {
        "cache.ttlInSeconds": 3600,
        "contentHandling": "CONVERT_TO_TEXT" // Ensure this matches the backend content type
      }
    }
  ]
}

Automate Testing:

Backend Service Tests:

PYTHON
import unittest

class TestBackendService(unittest.TestCase):
    def test_correct_content_type(self):
        # Mock request and expected response
        response = {"data": "example"}
        headers = {"Content-Type": "application/json"}

        self.assertEqual(headers.get("Content-Type"), "application/json")

Gateway Level Tests:

PYTHON
import unittest
from unittest.mock import patch

class TestGatewayIntegration(unittest.TestCase):
    @patch('requests.post')
    def test_response_headers(self, mock_post):
        # Mocked response from backend
        response = {"data": "example"}
        headers = {"Content-Type": "application/json"}

        mock_post.return_value.status_code = 200
        mock_post.return_value.headers = headers

        gateway_response = self.gatewayservice.get_data()
        self.assertEqual(gateway_response.headers['Content-Type'], 'application/json')

Integrate into CI/CD:

Ensure these tests run as part of your deployment pipeline, e.g., using Jenkins or GitHub Actions.

#### Closing Summary

By implementing robust content-type handling and automated regression checks, you can prevent future double-encoding issues in the API Gateway. This involves verifying backend responses during development and ensuring gateway configurations are correctly updated. Automating this process through CI/CD ensures that such issues do not slip into production.

#### Common Mistakes
and Follow-Ups
Overlooking Content-Type Mismatches: Always verify content types, especially when dealing with new deployments.
Improper Gateway Configuration: Double-check your gateway settings to ensure they handle all possible Content-Type values correctly.
Lack of Automated Tests: Ensure that both backend and gateway-level tests are in place and run as part of the CI/CD process.

By addressing these points, you can effectively manage content-type handling and avoid similar issues in the future.

