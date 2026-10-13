---
type: "practice"
difficulty: "Deep"
---

# How would you implement an A/B experiment across two model versions in FastAPI where user bucketing is sticky and the assignment plus outcome are logged for later analysis?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Service Architecture and Versioning](<../../../../TOOLS/Python/FastAPI/Service_Architecture_and_Versioning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a sticky A/B experiment with FastAPI, where users are assigned to specific model versions based on their unique identifiers (e.g., user IDs), we need to ensure that once a user is assigned, they remain in that assignment throughout the experiment. Additionally, we must log the outcome of each model version for analysis. This solution leverages FastAPI's flexibility and integrates with external storage systems like Redis or PostgreSQL for state management.

The core idea is to use a sticky session approach where the user ID determines their bucket (model version). We will also set up logging mechanisms to capture the assignment and outcomes, ensuring robustness and scalability.

**Code:**

```python
from fastapi import FastAPI, HTTPException, Depends, Path
import uuid
from typing import Optional
from pydantic import BaseModel

app = FastAPI()

# Simulated user data storage (e.g., Redis or PostgreSQL)
user_buckets = {}

class ExperimentOutcome(BaseModel):
    user_id: str
    model_version: int
    outcome: bool  # Example binary outcome

outcomes_db = []  # Simulating a database for outcomes

def get_user_bucket(user_id: str) -> int:
    if user_id not in user_buckets:
        user_buckets[user_id] = uuid.uuid4().int % 2  # Simple bucketing logic
    return user_buckets[user_id]

@app.post("/user/{user_id}/experiment/")
async def experiment(user_id: str, outcome: ExperimentOutcome):
    model_version = get_user_bucket(user_id)

    # Logging the assignment and outcome
    outcomes_db.append({"user_id": user_id, "model_version": model_version, **outcome.dict()})

    return {"message": f"User {user_id} assigned to model version {model_version}, Outcome: {outcome.outcome}"}
```

**Example input / output:**

```

Dry-run input:

POST /user/12345/experiment/ with payload:
JSON
Copy
{
  "user_id": "12345",
  "model_version": 0,
  "outcome": true
}

Expected output:

JSON
Copy
{
  "message": "User 12345 assigned to model version 0, Outcome: true"
}
```

**Data:**

```json
{
  "user_id": "12345",
  "model_version": 0,
  "outcome": true
}
```

