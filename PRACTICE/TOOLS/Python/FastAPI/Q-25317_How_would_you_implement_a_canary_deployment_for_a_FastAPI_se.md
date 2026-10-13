---
type: "practice"
difficulty: "Expert"
---

# How would you implement a canary deployment for a FastAPI service where you route 5% of traffic to the new version and automatically roll back on error-rate or latency regression?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limiting and Canary Releases](<../../../../TOOLS/Python/FastAPI/Rate_Limiting_and_Canary_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a canary deployment for a FastAPI service, we'll need to route 5% of incoming requests to the new version while keeping 95% on the old version. This approach allows us to gradually introduce changes and monitor their impact before fully rolling out to all users. The key steps involve setting up traffic routing based on request headers or query parameters, implementing error detection mechanisms, and automating the rollback process if issues are detected.

The problem can be broken down into several components:

Traffic Routing: Directing a portion of requests to the new version.
Error Detection: Monitoring error rates or latency for both versions.
Automated Rollback: Rolling back to the old version if errors exceed predefined thresholds.

Let's walk through these steps in detail:

#### Triaging the Question
The interviewer is probing our understanding of deployment strategies, traffic management, and automated rollback mechanisms.
We need to assume that we have both versions of the service deployed and can route requests between them based on certain criteria.
Common challenges include ensuring minimal disruption during the transition and maintaining robust monitoring.
Detailed Solution from First Principles

Traffic Routing

One common approach is to use a header or query parameter to differentiate between traffic. For example, adding ?version=old for old version requests and ?version=new for new version requests.
In FastAPI, we can create middleware to inspect these parameters and route the request accordingly.

Error Detection

We need metrics like HTTP error rates or latency to trigger a rollback if something goes wrong in the new version.
A simple way is to set up Prometheus or similar monitoring tools to track these metrics. For example, we can define a metric error_rate and check its value over time.

Automated Rollback

If error rates exceed a certain threshold (e.g., 5%), the new version should be rolled back.
This involves writing a script or using an orchestration tool like Kubernetes to manage service versions and their deployments.

#### Example
Implementation

First, let’s set up FastAPI with middleware for traffic routing:

PYTHON
from fastapi import FastAPI, Request
import os

app = FastAPI()

@app.middleware("http")
async def route_to_new_version(request: Request):
    # Check if version parameter is 'new'
    if request.query_params.get('version') == 'new':
        # Use the new service URL
        new_service_url = "http://localhost:8001/new"
        return await call_remote_service(new_service_url, request)

    # Default to old service
    old_service_url = "http://localhost:8000/old"
    return await call_remote_service(old_service_url, request)

async def call_remote_service(service_url, request):
    import requests
    response = await requests.request(
        method=request.method,
        url=service_url + request.url.path,
        headers=dict(request.headers),
        json=[await s for s in request.body],
        stream=request.stream)
    return response

# Run the FastAPI app
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)

Next, we need to monitor error rates and latency:

PYTHON
import prometheus_client
from prometheus_client import Counter

# Error counter for the new version
new_version_error_count = Counter('new_version_http_request_errors', 'HTTP Request Errors on New Version')

@app.middleware("http")
async def record_error(request: Request):
    if request.query_params.get('version') == 'new':
        try:
            await call_remote_service(service_url, request)
        except Exception as e:
            new_version_error_count.inc()
            raise

Finally, we set up an automated rollback mechanism:

PYTHON
import subprocess

def roll_back():
    # Restart the old version and stop the new version using orchestration tools
    subprocess.run(["kubectl", "rollout", "undo", "deployment/old-deployment"])
    subprocess.run(["kubectl", "scale", "deployment/new-deployment", "--replicas=0"])

# Check thresholds every minute and roll back if needed
import time

while True:
    current_error_rate = new_version_error_count.total / (time.time() - start_time)
    if current_error_rate > 5:  # Example threshold of 5%
        roll_back()
        break
    time.sleep(60)

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution provides a basic framework for implementing canary deployments with FastAPI. Key takeaways include using middleware to route traffic, leveraging monitoring tools like Prometheus, and automating rollbacks based on error rates.

#### Common mistakes
include not thoroughly testing the setup in a staging environment, overlooking edge cases such as network issues, or not setting up proper logging for debugging purposes.

#### Follow-ups
might involve discussing more advanced topics like A/B testing, blue-green deployments, or using specialized deployment tools.

**Code:**

```python
from fastapi import FastAPI, Request
import os

app = FastAPI()

@app.middleware("http")
async def route_to_new_version(request: Request):
    if request.query_params.get('version') == 'new':
        new_service_url = "http://localhost:8001/new"
        return await call_remote_service(new_service_url, request)

    old_service_url = "http://localhost:8000/old"
    return await call_remote_service(old_service_url, request)

async def call_remote_service(service_url, request):
    import requests
    response = await requests.request(
        method=request.method,
        url=service_url + request.url.path,
        headers=dict(request.headers),
        json=[await s for s in request.body],
        stream=request.stream)
    return response

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)

import prometheus_client
from prometheus_client import Counter

new_version_error_count = Counter('new_version_http_request_errors', 'HTTP Request Errors on New Version')

@app.middleware("http")
async def record_error(request: Request):
    if request.query_params.get('version') == 'new':
        try:
            await call_remote_service(service_url, request)
        except Exception as e:
            new_version_error_count.inc()
            raise

def roll_back():
    subprocess.run(["kubectl", "rollout", "undo", "deployment/old-deployment"])
    subprocess.run(["kubectl", "scale", "deployment/new-deployment", "--replicas=0"])

import time

while True:
    current_error_rate = new_version_error_count.total / (time.time() - start_time)
    if current_error_rate > 5:
        roll_back()
        break
    time.sleep(60)
```

