---
type: "practice"
difficulty: "Deep"
---

# How would you set up a Python canary rollout for a new prompt or model version in a GenAI service, comparing live faithfulness and latency before promoting to 100%?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retries with Deadline Budgets](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/Retries_with_Deadline_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a Python-based canary rollout for a GenAI service, we need to carefully manage the deployment of new prompt or model versions while ensuring that any issues are caught early. The goal is to monitor both faithfulness and latency before fully promoting the new version. This involves setting up controlled, incremental deployments, monitoring key metrics, and having robust logging and alerting systems in place.

#### Triaging the Question

The core of this question revolves around understanding how to implement a canary release strategy for an AI service using Python. The interviewer is probing your knowledge of deployment strategies, monitoring tools, and best practices for maintaining service quality and performance.

#### Assumptions

We have access to version control systems (like Git) for managing different versions of the model or prompt.
There are existing logging and alerting mechanisms in place within our infrastructure.
The new version is ready but needs validation before full deployment.

#### First Principles Solution

Version Management:

Use a versioned approach where you have both the current stable (production) and the new (canary) versions of your AI service running simultaneously.

Traffic Splitting:

Implement a mechanism to route a portion of incoming requests to the canary version while the rest go to the production version.

Monitoring Setup:

Set up metrics to monitor both faithfulness and latency for the canary version.
Define thresholds for acceptable performance levels.

Automated Logging and Alerting:

Log detailed information about each request, including timestamps, input data, output responses, and any errors or warnings.
Configure alerts when certain thresholds are exceeded.

Health Checks:

Periodically check the health of both versions to ensure they meet predefined criteria.

Promotion Strategy:

Based on the results from monitoring, decide whether to promote the canary version to 100% or revert it back to the production version.
Detailed Implementation
Version Management

First, we need a way to manage different versions of our service. We can use environment variables or configuration files to control which version is being served based on some identifier (e.g., CANARY_VERSION).

PYTHON
import os

def get_model_version():
    return os.environ.get('CANARY_VERSION', 'production')
Traffic Splitting

We need a way to split the traffic between the two versions. A common approach is to use a simple hash-based or round-robin algorithm.

PYTHON
import hashlib

def choose_version(request_id):
    version = get_model_version()
    if version == 'canary':
        # Hash request ID and use modulo to decide canary percentage
        return (hashlib.md5(request_id.encode()).hexdigest()[-1] % 10) < 3
    return False
Monitoring Setup

We need to define metrics for both faithfulness and latency. For simplicity, let's assume we have a Response class that holds the response details.

PYTHON
class Response:
    def __init__(self, content, timestamp):
        self.content = content
        self.timestamp = timestamp

def measure_latency(response):
    return (datetime.now() - response.timestamp).total_seconds()

def check_faithfulness(response, expected_output):
    return Levenshtein.distance(response.content, expected_output) < threshold
Automated Logging and Alerting

We can use libraries like logging for basic logging and third-party services like Prometheus or Datadog for more advanced monitoring.

PYTHON
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

handler = logging.StreamHandler()
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)

def log_response(request_id, response):
    logger.info(f"Request ID: {request_id}, Response Content: {response.content}")
Health Checks

We can periodically check the health of both versions by sending test requests.

PYTHON
import time

def perform_health_check(version):
    # Send a test request and measure latency and faithfulness
    start_time = datetime.now()
    response = send_test_request(version)
    elapsed_time = (datetime.now() - start_time).total_seconds()

    if check_faithfulness(response.content, expected_output) and elapsed_time < max_latency:
        return True
    else:
        return False

def send_test_request(version):
    # Send a test request to the specified version and get response
    pass  # Implement this function based on your service's architecture

# Check both versions every hour
while True:
    if perform_health_check('canary'):
        logger.info("Canary version is healthy")
    else:
        logger.warning("Canary version is unhealthy, reverting to production")

    time.sleep(3600)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, setting up a canary rollout involves managed traffic splitting, detailed monitoring, automated logging, and periodic health checks. The key mistakes to avoid include insufficient testing, inadequate alerting, and failing to handle edge cases (e.g., high latency during peak times).

Common follow-ups might involve discussing specific tools used for logging/alerting, the architecture of your service, or trade-offs between different deployment strategies.

**Comparison:**

```
Version	Health Status
Canary	Healthy
Production	Healthy
Request ID	Response Content	Timestamp
12345	New model response	2023-10-01T12:00:00Z
```

**Code:**

```python
import os
import hashlib
from datetime import datetime

def get_model_version():
    return os.environ.get('CANARY_VERSION', 'production')

def choose_version(request_id):
    version = get_model_version()
    if version == 'canary':
        return (hashlib.md5(request_id.encode()).hexdigest()[-1] % 10) < 3
    return False

class Response:
    def __init__(self, content, timestamp):
        self.content = content
        self.timestamp = timestamp

def measure_latency(response):
    return (datetime.now() - response.timestamp).total_seconds()

def check_faithfulness(response, expected_output):
    return Levenshtein.distance(response.content, expected_output) < threshold

def log_response(request_id, response):
    logger.info(f"Request ID: {request_id}, Response Content: {response.content}")

def perform_health_check(version):
    start_time = datetime.now()
    response = send_test_request(version)
    elapsed_time = (datetime.now() - start_time).total_seconds()

    if check_faithfulness(response.content, expected_output) and elapsed_time < max_latency:
        return True
    else:
        return False

def send_test_request(version):
    # Send a test request to the specified version and get response
    pass  # Implement this function based on your service's architecture

# Check both versions every hour
while True:
    if perform_health_check('canary'):
        logger.info("Canary version is healthy")
    else:
        logger.warning("Canary version is unhealthy, reverting to production")

    time.sleep(3600)
```

**Example input / output:**

```text
Input:
request_id = "12345"

Output:
Request ID: 12345, Response Content: New model response
```

**Data:**

```yaml
- request_id: "12345"
  response_content: "New model response"
  timestamp: "2023-10-01T12:00:00Z"
- expected_output: "Expected response from old model"
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Choose Version
6. Canary Version?
7. Log Response
8. Measure Latency
9. Decide to Promote?
10. Promote Canary to 100%
11. Revert to Production
12. Serve from Production

