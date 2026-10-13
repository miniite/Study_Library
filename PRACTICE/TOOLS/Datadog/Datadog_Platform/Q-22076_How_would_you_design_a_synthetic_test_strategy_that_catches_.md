---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a synthetic test strategy that catches an endpoint outage before real users notice while avoiding false alarms from transient network blips?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Alert Automation and Runbooks](<../../../../TOOLS/Datadog/Datadog_Platform/Alert_Automation_and_Runbooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust synthetic test strategy using Datadog, we need to ensure that we can catch an endpoint outage early without generating unnecessary false positives due to temporary network issues or transients. Here’s how you would approach this:

#### Overview
We'll use Datadog's Synthetic Monitoring feature to create tests that simulate real user behavior and monitor the availability of our endpoints. The key is to implement a strategy that distinguishes between genuine outages and transient errors.

#### Triaging the Question

What the Question Is Asking: Design a synthetic test strategy that can detect when an endpoint goes down before it affects actual users.
What’s Being Probed: The ability to write effective tests, understanding monitoring patterns, and handling false positives.

#### Assumptions
Assume you have access to Datadog's Synthetic Monitoring capabilities.

#### First-Principles Solution

Test Types: Implement both HTTP checks (to verify endpoint reachability) and custom steps (for more detailed checks like specific API responses).
Frequency of Tests: Schedule tests at intervals that are frequent enough to catch outages early but not so frequent as to generate false alarms.
Error Handling: Implement retries with exponential backoff to handle temporary network blips. Use the ‘tolerance’ feature in Datadog Synthetic Monitoring to set a threshold for acceptable error rates.
Detection Logic: Set up alerts that trigger only when errors persist over multiple test runs, ensuring transient issues are ignored.

Detailed Steps:

Step 1: Define Tests

MARKDOWN
| Test Name | Endpoint URL | Method (HTTP/HTTPS) | Expected Response Code |
|-----------|--------------|---------------------|-----------------------|
| Home Page | https://example.com | GET             | 200                  |
| API Endpt | https://api.example.com/v1/data | POST   | 200                 |

Step 2: Schedule Tests

MARKDOWN
| Test Name | Schedule (mins) |
|-----------|-----------------|
| Home Page | 5               |
| API Endpt | 30              |

Step 3: Implement Retries and Tolerance

Use Datadog's retry mechanism to handle transient errors. Configure the tests with a backoff strategy like 1,2,4,8 minutes.
Set tolerance thresholds for each test (e.g., 5 retries allowed in 5 minutes).

Step 4: Alerting Strategy

Alerts should only trigger if an error persists over multiple test runs. This can be configured using the ‘tolerance’ feature to set a threshold like “alert after 2 out of 3 consecutive failures”.
Ensure alerts are sent via channels such as email, Slack, or PagerDuty for immediate action.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Summary: By carefully configuring synthetic tests in Datadog and setting up robust error handling and alerting mechanisms, we can effectively detect endpoint outages while minimizing false positives.

#### Common Mistakes
Not defining clear retry policies or tolerance thresholds.
Setting test schedules too frequently to catch outages but generating too many false alarms.
Ignoring transient network issues in the alerting strategy.

By following this approach, you can ensure that your synthetic tests are both sensitive enough to detect real outages and robust enough to handle temporary network blips effectively. This will help maintain high availability of endpoints while reducing noise from false positives.

**Comparison:**

| Test Name | Endpoint URL | Method (HTTP/HTTPS) | Expected Response Code |
|-----------|--------------|---------------------|-----------------------|
| Home Page | https://example.com | GET             | 200                  |
| API Endpt | https://api.example.com/v1/data | POST   | 200                 |
| Test Name | Schedule (mins) |
|-----------|-----------------|
| Home Page | 5               |
| API Endpt | 30              |

**Code:**

```python
# Example Python code for setting up Datadog Synthetic tests with retry policy
from datadog import initialize, SYNTHETIC_API_KEY

initialize(api_key=SYNTHETIC_API_KEY)

def create_test(name, url, method='GET', expected_status_code=200):
    return {
        'name': name,
        'type': 'http',
        'options': {
            'method': method,
            'url': url,
            'expected_status_codes': [str(expected_status_code)],
            'tolerance': 5,  # Allow up to 5 retries in 5 minutes
            'retry_strategy': {'attempts': 4, 'backoff_factor': 2}
        }
    }

# Create and schedule tests
test_home_page = create_test('Home Page', 'https://example.com')
test_api_endpt = create_test('API End Point', 'https://api.example.com/v1/data')

tests = [test_home_page, test_api_endpt]

for test in tests:
    response = SYNTHETIC_API.create(test)
    print(response)
```

**Example input / output:**

```text
graph LR;
    A[Create Test Home Page] --> B{Is Error Tolerated?};
    B -- Yes --> C[Schedule Test Every 5 Minutes];
    B -- No --> D[Retry After Error];
    C --> E[Alert if Persistent Error];
    D --> F[Handle Temporary Error];

    A2[Create Test API End Point] --> G{Is Error Tolerated?};
    G -- Yes --> H[Schedule Test Every 30 Minutes];
    G -- No --> I[Retry After Error];
    H --> J[Alert if Persistent Error];
    I --> K[Handle Temporary Error];

    B2 --> C;
    G2 --> H;
```

**Data:**

```yaml
# Example YAML for Datadog Synthetic test configurations
- name: Home Page
  type: http
  options:
    method: GET
    url: https://example.com
    expected_status_codes: [200]
    tolerance: 5
    retry_strategy:
      attempts: 4
      backoff_factor: 2

- name: API End Point
  type: http
  options:
    method: POST
    url: https://api.example.com/v1/data
    expected_status_codes: [200]
    tolerance: 5
    retry_strategy:
      attempts: 4
      backoff_factor: 2
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. No
6. Yes
7. No
8. Yes
9. Start
10. Define Test Cases
11. Is Error Tolerated?
12. Retry After Error
13. Schedule Tests
14. Persisting Error?
15. Continue Monitoring
16. Trigger Alert
17. Create Test Home Page
18. Is Error Tolerated?
19. Retry After Error
20. Schedule Test Every 5 Minutes
21. Alert if Persistent Error
22. Create Test API End Point
23. Is Error Tolerated?
24. Retry After Error
25. Schedule Test Every 30Minutes
26. Alert if Persistent Error

