---
type: "practice"
difficulty: "Deep"
---

# How would you design a FastAPI cost dashboard endpoint that aggregates token spend by model, tenant, and endpoint from your logged LLM calls in near real time?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limiting and Canary Releases](<../../../../TOOLS/Python/FastAPI/Rate_Limiting_and_Canary_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a FastAPI cost dashboard endpoint, we need to aggregate token spends by model, tenant, and endpoint in near real-time. This involves setting up efficient data collection, storage, and querying mechanisms within our API. We'll break down the solution step-by-step from first principles.

Firstly, let's define what "near real time" means for our context. For this dashboard, we can aim for a latency of around 1-2 seconds to reflect any changes in token spend accurately. This requires us to have an efficient data pipeline that updates the cost metrics regularly and a FastAPI endpoint that queries these metrics.

**Comparison:**

| Tenant | Model   | Endpoint |
|--------|---------|----------|
| 1      | model1  | endpointA|
| 2      | model2  | endpointB|

**Code:**

```python
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from datetime import datetime

app = FastAPI()

# Dummy database models for demonstration purposes
class Tenant:
    def __init__(self, id, name):
        self.id = id
        self.name = name

class Model:
    def __init__(self, id, name):
        self.id = id
        self.name = name

class Endpoint:
    def __init__(self, id, model_id, tenant_id, name):
        self.id = id
        self.model_id = model_id
        self.tenant_id = tenant_id
        self.name = name

# Dummy database session (in a real application, use SQLAlchemy or another ORM)
session_db = []

def get_db():
    db = session_db  # Assume this is an actual DB session
    try:
        yield db
    finally:
        pass

@app.get("/cost-dashboard")
async def cost_dashboard(model_id: int = None, tenant_id: int = None, endpoint_name: str = None, db: Session = Depends(get_db)):
    start_time = datetime.now()
    result = aggregate_costs(db, model_id, tenant_id, endpoint_name)
    end_time = datetime.now()
    print(f"Query time: {end_time - start_time}")
    return {"costs": result}
```

**Example input / output:**

```text
Request:
GET /cost-dashboard?model_id=1&tenant_id=1&endpoint_name=endpointA

Response:
{
  "costs": [
    {
      "model_id": 1,
      "tenant_id": 1,
      "endpoint_name": "endpointA",
      "token_spend": 500
    }
  ]
}
```

**Data:**

```

[ { "id": 1, "name": "tenant1" }, { "id": 2, "name": "tenant2" } ]

CODE
Copy

```json
[
  {
    "id": 1,
    "name": "model1"
  },
  {
    "id": 2,
    "name": "model2"
  }
]
JSON
Copy
[
  {
    "id": 1,
    "model_id": 1,
    "tenant_id": 1,
    "name": "endpointA"
  },
  {
    "id": 2,
    "model_id": 1,
    "tenant_id": 2,
    "name": "endpointB"
  }
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is a query parameter provided?
5. Retrieve data from database
6. Are the parameters valid?
7. Fetch aggregated costs usingparameters
8. Return error response
9. Generate and return costdashboard JSON
10. End

