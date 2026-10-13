---
type: "practice"
difficulty: "Expert"
---

# How would you design a FastAPI endpoint to enforce a per-org quota (e.g. 1M API calls/month) accurately without a hot counter becoming a write bottleneck?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limiting and Canary Releases](<../../../../TOOLS/Python/FastAPI/Rate_Limiting_and_Canary_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a FastAPI endpoint that enforces a per-organization quota, we need to ensure accuracy in tracking and enforcing the rate limits while minimizing the risk of a hot counter becoming a write bottleneck. This involves leveraging distributed systems concepts like leader election or shared database transactions.

First, let's triage the question: The interviewer is probing your understanding of handling stateful operations across multiple API calls, ensuring scalability, and avoiding single points of failure in a distributed environment. We need to assume that each organization has a unique identifier (e.g., org_id), and we want to limit the number of API calls per month for each organization.

Core Solution & Key Idea

The key idea is to use a leader election strategy or distributed counters like Redix or distributed transactions to safely track and enforce quotas without creating a bottleneck. This approach ensures that quota enforcement can scale horizontally, distributing load across multiple instances.

First-Principles Reasoning
Identify the Problem: We need to accurately count API calls per organization.
Stateless vs Stateful Operations: FastAPI is stateless by design, making it challenging to maintain state across requests. However, we can use a combination of leader election and distributed counters to achieve our goal.
Leader Election: Each instance should have the authority to update the quota count for its organization once it has been elected as the leader. This avoids contention on a single write operation.
Distributed Counters: Use a distributed counter service like Redis or an external service that manages these counters in a scalable manner.
Detailed Solution

Define the Quota Model:

PYTHON
class QuotaModel(BaseModel):
    org_id: str
    calls_this_month: int = 0

Leader Election Strategy: Use a distributed consensus algorithm like Raft or Paxos to elect a leader among API instances for each organization. The leader is responsible for updating the quota count.

Increment Quota on Leader:

PYTHON
@app.get("/api/v1/quota/increment/{org_id}")
async def increment_quota(org_id: str):
    # Check if current instance is the leader for this org_id
    if is_leader_for_org_id(org_id):
        with session.begin():
            quota_model = session.query(QuotaModel).filter_by(org_id=org_id).first()
            if not quota_model:
                quota_model = QuotaModel(org_id=org_id)
                session.add(quota_model)
            quota_model.calls_this_month += 1
            session.commit()
            return {"status": "success", "calls_this_month": quota_model.calls_this_month}
    else:
        raise HTTPException(status_code=403, detail="Not authorized to update quota")

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we use a leader election strategy combined with distributed counters to enforce per-organization quotas accurately without causing write bottlenecks. Key mistakes include not having a fail-safe mechanism for leader elections or relying solely on local state updates.

Common follow-ups could involve discussing the trade-offs between different consensus algorithms, how to handle network partitions, and what backup strategies exist in case of leader failure.

**Comparison:**

```
Operation	Description
is_leader_for_org_id	Checks if the current instance is elected as the leader for a given org_id. This function might involve querying a distributed key-value store or consensus service.
increment_quota	Increments the quota count by 1 and updates the database only if the current instance is the leader.
```

**Code:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.asyncio import AsyncSession

app = FastAPI()

class QuotaModel(BaseModel):
    org_id: str
    calls_this_month: int = 0

# Mock database setup (for illustration)
engine = create_async_engine("sqlite+aiosqlite:///test.db")
AsyncSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine, class_=AsyncSession)

@app.get("/api/v1/quota/increment/{org_id}")
async def increment_quota(org_id: str):
    # Check if current instance is the leader for this org_id
    if is_leader_for_org_id(org_id):
        async with AsyncSessionLocal() as session:
            quota_model = await session.query(QuotaModel).filter_by(org_id=org_id).first()
            if not quota_model:
                quota_model = QuotaModel(org_id=org_id)
                await session.add(quota_model)
            quota_model.calls_this_month += 1
            await session.commit()
            return {"status": "success", "calls_this_month": quota_model.calls_this_month}
    else:
        raise HTTPException(status_code=403, detail="Not authorized to update quota")
```

**Example input / output:**

```text
// Sample input/output:
{
    "org_id": "example_org"
}
Response:
{
    "status": "success",
    "calls_this_month": 1
}
```

**Data:**

```sql
CREATE TABLE QuotaModel (
   org_id VARCHAR(255) PRIMARY KEY,
   calls_this_month INT DEFAULT 0
);
```

**Flow:**

1. Yes
2. No
3. API Call
4. Is Current Instance Leader?
5. Increment Quota Count
6. Commit Transaction
7. Return Success
8. 403 Forbidden

