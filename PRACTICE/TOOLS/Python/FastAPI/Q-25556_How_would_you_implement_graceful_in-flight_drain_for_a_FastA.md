---
type: "practice"
difficulty: "Deep"
---

# How would you implement graceful in-flight drain for a FastAPI GPU service during a Kubernetes rolling deploy so mid-inference requests are not killed and the GPU frees cleanly?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limiting and Canary Releases](<../../../../TOOLS/Python/FastAPI/Rate_Limiting_and_Canary_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that mid-inference requests are not terminated and the GPU resources are freed gracefully during a Kubernetes rolling deployment of a FastAPI GPU service, we need to implement an in-flight drain mechanism. This involves several key steps:

TL;DR: We will use Kubernetes annotations and custom FastAPI middleware to manage incoming requests during a rolling update. The service will be marked as ready after receiving a signal from the controller that it can handle traffic again.

#### Triage
The core of the question is about gracefully handling ongoing requests when a service undergoes a rolling deployment in Kubernetes, ensuring no mid-inference requests are terminated abruptly and making sure GPU resources are released cleanly.

Step-by-Step Solution:

Kubernetes Annotations: Add a custom annotation to the deployment manifest that signals the controller to perform a drain operation.
FastAPI Middleware: Implement middleware in FastAPI to manage incoming requests during the drain period, ensuring they complete before the service is fully terminated.
Service Controller: Use a Kubernetes controller to monitor the state of the service and initiate graceful shutdown based on annotations.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Ensure that mid-inference requests are not killed by marking the service as ready after completing ongoing operations.
Gracefully release GPU resources when the service is no longer handling traffic.

#### Common mistakes
include terminating services too quickly, which can lead to data corruption or incomplete processing. Always ensure a proper signal mechanism.

**Comparison:**

| Time | Event                  |
|------|------------------------|
| T0   | Request 1 received     |
| T2   | Rolling update starts  |
| T3   | Service marks itself not ready for new requests |
| T4   | Middleware handles request completion             |
| T5   | Service is marked as ready again                  |

**Code:**

```python
# Example FastAPI middleware for graceful shutdown

from fastapi import APIRouter, Request
import asyncio
import logging

router = APIRouter()
logger = logging.getLogger(__name__)

@router.middleware("http")
async def handle_in_flight_requests(request: Request, call_next):
    if request.state.in_drain:
        # Wait for in-flight requests to complete
        await asyncio.sleep(1)  # Simulate a delay; actual logic would be more complex

    response = await call_next(request)
    return response
```

**Example input / output:**

```text
Request 1 (In-flight): Received at time T0, expected completion by time T1.
Rolling Update Initiated at T2.
Service marked as not ready for new requests at T3.
Middleware handles request completion between T3 and T4.
Service marked as ready again at T5.
```

**Data:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-fastapi-gpu-service
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: my-fastapi-gpu-service
    spec:
      containers:
      - name: fastapi-container
        image: my-fastapi-image
        args:
        - --inflight-drain-annotation=k8s.io/ingress.class=nginx
```

**Flow:**

1. Yes
2. Rolling Update Starts
3. Kubernetes Controller DetectsChange
4. Service Marks Self Not Ready
5. Middleware Handles In-flightRequests
6. Requests Complete
7. Service Marks Self Ready Again
8. Deployment Complete

