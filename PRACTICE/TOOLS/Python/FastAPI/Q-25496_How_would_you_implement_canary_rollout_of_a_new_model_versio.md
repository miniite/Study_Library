---
type: "practice"
difficulty: "Deep"
---

# How would you implement canary rollout of a new model version in FastAPI where 5% of traffic is scored by the candidate and you auto-roll-back on a live-accuracy or latency regression?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limiting and Canary Releases](<../../../../TOOLS/Python/FastAPI/Rate_Limiting_and_Canary_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to implement a canary rollout strategy using FastAPI that allows us to deploy a new model version while only 5% of the traffic is routed through it. The system should automatically rollback if there's a live-accuracy or latency regression compared to the current stable version.

To achieve this, we'll leverage FastAPI for its simplicity and flexibility, along with some custom middleware and database operations to monitor performance metrics. We assume that the new model and the old one are both deployed as separate endpoints in FastAPI, and we use a database to keep track of the traffic split and any issues encountered.

#### Triage

The question is essentially about implementing a canary release mechanism using FastAPI. The core aspects being probed include:

Traffic Splitting: How to route 5% of the requests to the new model while keeping the rest with the old version.
Performance Metrics Collection: How to collect and compare performance metrics such as accuracy and latency between the two versions.
Rollback Mechanism: What criteria should trigger a rollback, and how to implement this in FastAPI.

#### Assumptions
The new model is deployed alongside the current stable version with different endpoints.
We use a database (e.g., PostgreSQL) to log traffic splits and performance metrics.
The system checks for live-accuracy or latency regressions using predefined thresholds.
The rollback process is initiated by comparing real-time performance data against historical data.

#### First Principles Solution

Traffic Splitting

Use FastAPI middleware to intercept requests and route them based on a random number generator that selects 95% of the traffic for the old model and 5% for the new.

Performance Metrics Collection

Use metrics libraries like Prometheus or Datadog to monitor accuracy and latency for both models.
Log these metrics into a database with timestamps, model version, and other relevant metadata.

Rollback Mechanism

Define thresholds for live-accuracy (e.g., 95% of the old version) and latency (e.g., within 100ms).
Implement a real-time monitoring system that compares current metrics against historical data.
If any metric crosses the threshold, trigger an automatic rollback.
Detailed Implementation
PYTHON
from fastapi import FastAPI, Depends, Request
import random
from sqlalchemy.orm import Session

app = FastAPI()

# Database dependency
def get_db():
    db = None
    try:
        db = sessionmaker()  # Assuming a database session factory is defined elsewhere
        yield db
    finally:
        db.close()

@app.middleware("http")
async def canary_middleware(request: Request, call_next):
    model_version = "stable"  # Default to stable version

    if random.random() < 0.05:  # Route 5% of traffic to the candidate
        model_version = "candidate"

    response = await call_next(request)

    # Log traffic split and performance metrics
    with Session(get_db()) as db:
        log_traffic(db, request.path, model_version)

    return response

def log_traffic(db: Session, path: str, model_version: str):
    metric_data = get_metrics(path)  # Fetch or calculate relevant metrics
    if model_version == "candidate":
        compare_and_log(db, metric_data)

@app.get("/stable-model")
async def stable_model():
    return {"model": "stable", "response": "This is the old version"}

@app.get("/candidate-model")
async def candidate_model():
    return {"model": "candidate", "response": "This is the new version"}

#### Closing Summary
and Common Mistakes
Ensure robust logging of traffic splits and metrics.
Carefully define rollback criteria to avoid unnecessary rollbacks due to noise in performance data.
Regularly review and adjust thresholds based on real-world usage patterns.
Common Follow-Ups
How do you handle edge cases where the new model is not ready?
What if there are multiple canary versions? How would you manage them?

**Comparison:**

```
Model Version	Response
Stable	This is the old version
Candidate	This is the new version
```

**Code:**

```python
from fastapi import FastAPI, Depends, Request
import random
from sqlalchemy.orm import Session

app = FastAPI()

def get_db():
    db = None
    try:
        db = sessionmaker()  # Assuming a database session factory is defined elsewhere
        yield db
    finally:
        db.close()

@app.middleware("http")
async def canary_middleware(request: Request, call_next):
    model_version = "stable"  # Default to stable version

    if random.random() < 0.05:  # Route 5% of traffic to the candidate
        model_version = "candidate"

    response = await call_next(request)

    with Session(get_db()) as db:
        log_traffic(db, request.path, model_version)

    return response

def log_traffic(db: Session, path: str, model_version: str):
    metric_data = get_metrics(path)  # Fetch or calculate relevant metrics
    if model_version == "candidate":
        compare_and_log(db, metric_data)
```

**Example input / output:**

```text
Input:
GET /stable-model

Output (for 95% of requests):
{"model": "stable", "response": "This is the old version"}

Input:
GET /candidate-model

Output (for 5% of requests):
{"model": "candidate", "response": "This is the new version"}
```

**Data:**

```json
{
  "stable_model_response": {"model": "stable", "response": "This is the old version"},
  "candidate_model_response": {"model": "candidate", "response": "This is the new version"}
}
```

**Flow:**

1. Yes
2. No
3. True
4. False
5. Request
6. Is request in canary group?
7. Route to candidate model
8. Route to stable model
9. Metric logging & comparison
10. Rollback if criteria met?
11. Roll back to stable version
12. Continue monitoring

