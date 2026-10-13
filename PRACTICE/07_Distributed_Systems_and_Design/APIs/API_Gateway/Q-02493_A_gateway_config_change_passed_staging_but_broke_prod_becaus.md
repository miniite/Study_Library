---
type: "practice"
difficulty: "Deep"
---

# A gateway config change passed staging but broke prod because prod has a header prod-only middleware sets; how do you build config testing with production-representative traffic replay to catch environment-specific breakage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Config Testing and Rollout](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Config_Testing_and_Rollout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that our API Gateway configuration changes are thoroughly tested in a way that accurately reflects their impact on the production environment, particularly focusing on middleware behavior that is environment-specific. The key challenge here is to create a robust testing framework that can simulate production traffic and replay it against staging or other non-production environments.

Triaging the Problem

Understanding the Issue: A recent configuration change in our API Gateway passed successfully through staging but broke the production system. This indicates there might be subtle differences between how staging and production behave, especially with respect to middleware.

What We Are Probing:

Environment-specific behavior of middlewares (like a prod-only header).
The ability to accurately simulate production traffic in non-production environments.

#### Assumptions

Our API Gateway is currently using staging as the default environment for testing.
We have access to detailed logs or metrics from production that can help us understand the impact of the configuration changes.

#### First-Principles Solution

Environment-Specific Traffic Simulation: The first step is to capture and analyze production traffic patterns and headers in a way that accurately simulates them in non-production environments. This involves setting up a mechanism for replaying production traffic through our staging or test environment.

Middleware Agnostic Testing Framework:

Develop a middleware-agnostic testing framework that can inject specific headers, cookies, and other metadata to mimic the production environment.
Ensure this framework can be easily configured to emulate different scenarios (e.g., prod-only headers, user roles).

Automated Testing: Integrate automated testing into our CI/CD pipeline so that every configuration change is automatically tested against a simulated production environment.

Monitoring and Logging:

Set up detailed logging in the staging environment to capture how different configurations behave.
Use A/B testing techniques to compare the behavior of the same request under different conditions (staging vs. prod).
Detailed Steps

Capture Production Traffic:

Deploy a lightweight agent or use existing tools like Wireshark, Burp Suite, or Fiddler to capture HTTP requests and responses.
Store this captured traffic in a format that can be easily replayed, such as HAR files (HTTP Archive).

Develop Replay Mechanism:

Use a middleware or a custom solution to inject captured headers and cookies into incoming requests.
Ensure the replay mechanism is configurable so we can specify which headers or cookies should be injected.

Middleware Agnostic Configuration:

Create a set of configuration profiles that map production-specific middleware behavior to non-production environments.
Implement logic in our API Gateway that allows us to switch between different configurations based on environment variables or tags.

Automated Test Execution:

Use tools like JMeter, Gatling, or custom scripts to run the captured traffic through the staging gateway with the appropriate configuration.
Monitor and log the results to identify any discrepancies compared to production behavior.

Continuous Monitoring and Feedback:

Set up alerts for critical failures during testing so we can address issues promptly.
Use A/B testing techniques to compare prod vs. staging behavior over time.

#### Closing Summary

By implementing a comprehensive traffic replay and configuration testing framework, we can significantly reduce the risk of environment-specific breakage when deploying changes to our API Gateway. This approach ensures that we accurately simulate production behavior in non-production environments, allowing us to catch issues early before they impact users.

#### Common Mistakes/Follow-Ups

Failing to capture enough traffic details: Ensure you have a comprehensive set of data points.
Not configuring the test environment correctly: Double-check the middleware configurations.
Relying solely on manual testing: Implement automation wherever possible to save time and reduce human error.

**Comparison:**

```
Environment	Header	Value
Staging	Authorization	Bearer abcdef123
Production	X-Custom-Header	custom-value
```

**Code:**

```python
# Example Python script for replaying production traffic in staging
import requests

def replay_traffic(har_file_path, staging_gateway_url):
    with open(har_file_path, 'r') as har_file:
        har_data = json.load(har_file)

    for entry in har_data['log']['entries']:
        headers = entry['request']['headers']
        cookies = entry.get('cookies', [])
        url = entry['request']['url']

        # Injecting headers and cookies into the request
        response = requests.get(url, headers=headers, cookies=cookies)
        print(f"Request to {url} with status code: {response.status_code}")
```

**Example input / output:**

```text
Input:
Replay traffic from file production_traffic.har against the staging gateway at https://staging.example.com/api/v1/data

Output:
Request to https://example.com/api/v1/data with status code: 200
Headers:
Content-Type: application/json
Authorization: Bearer abcdef123
X-Custom-Header: custom-value
Cookies:
session_id: abcd1234
```

**Data:**

```json
{
  "log": {
    "version": "1.2",
    "creator": {"name": "Python HAR Exporter", "version": "0.1"},
    "entries": [
      {
        "request": {
          "method": "GET",
          "url": "https://example.com/api/v1/data",
          "headers": [
            {"name": "Content-Type", "value": "application/json"},
            {"name": "Authorization", "value": "Bearer abcdef123"}
          ],
          "cookies": [{"name": "session_id", "value": "abcd1234"}]
        },
        "response": {
          "status": 200,
          "headers": [
            {"name": "Content-Type", "value": "application/json"},
            {"name": "X-Custom-Header", "value": "custom-value"}
          ],
          "content": {"text": "{'data': 'example_data'}"}
        }
      }
    ]
  }
}
```

**Flow:**

1. Capture Production Traffic
2. Store in HAR Format
3. Configure MiddlewareAgnostic Testing Framework
4. Replay Production Traffic inStaging
5. Monitor and Log Results

