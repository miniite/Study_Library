---
type: "practice"
difficulty: "Expert"
---

# How would you design idempotent request handling in FastAPI for a create-order endpoint so retries from an unreliable mobile client never double-charge, including where you store and expire the idempotency keys?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batching LLM Requests](<../../../../TOOLS/Python/FastAPI/Batching_LLM_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that idempotent requests are handled correctly in a FastAPI application, especially when dealing with a create-order endpoint from an unreliable mobile client, we need to design a system where each request can be safely retried without causing duplicate charges. The key is to generate and store unique idempotency keys for each request and ensure they expire after a certain period.

#### Triaging the Question

The core of this question lies in ensuring that:

Idempotency: Each request, even if retried multiple times, results in the same action being performed.
Avoiding Double-Charge: Ensuring no duplicate orders are created when requests are retried due to network issues or client crashes.

#### First-Principles Solution
Key Components:
Idempotency Keys Generation and Storage:
Generate a unique idempotency key for each request.
Store the key in a database with an associated timestamp to track when the request was made.
Expiry Mechanism:
Define a reasonable expiry time after which old requests can be safely ignored.

#### Implementation
Steps:
Generate Idempotency Key:
Use a combination of UUID, timestamps, or other unique identifiers to ensure each request has a distinct idempotency key.
Store in Database:
Save the idempotency key and timestamp in a database table specifically designed for idempotency tracking.
Check Expiry Before Processing:
Before processing the request, check if the idempotency key has expired based on the stored timestamp.

#### Example
Implementation
PYTHON
from fastapi import FastAPI, HTTPException, status
import uuid
from datetime import datetime, timedelta
from pydantic import BaseModel
from typing import Optional
from sqlalchemy.orm import Session
from your_database_setup import Base, IdempotencyKey  # Assuming a SQLAlchemy ORM setup

app = FastAPI()

class CreateOrderRequest(BaseModel):
    order_id: str
    product_name: str
    quantity: int

@app.post("/create-order")
async def create_order(request: CreateOrderRequest, db_session: Session):
    idempotency_key = uuid.uuid4()
    # Check if the key already exists and has not expired
    existing_key = db_session.query(IdempotencyKey).filter_by(key=idempotency_key).first()
    if existing_key and (datetime.utcnow() - existing_key.timestamp) < timedelta(minutes=5):
        raise HTTPException(status_code=status.HTTP_409_CONFLICT, detail="Request already processed")

    # Process the order
    db_session.add(IdempotencyKey(key=idempotency_key, timestamp=datetime.utcnow()))
    db_session.commit()

    # Simulate order creation
    result = {"order_id": request.order_id, "status": "created"}
    return result

# Example SQLAlchemy ORM setup for IdempotencyKey
class IdempotencyKey(Base):
    __tablename__ = "idempotency_keys"
    key = db.Column(db.String, primary_key=True)
    timestamp = db.Column(db.DateTime, default=datetime.utcnow)

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to properly store or check idempotency keys.
Not defining an appropriate expiry time for the keys, leading to potential conflicts.
Ignoring edge cases such as race conditions during key checks.

#### Follow-Ups

Discussing how this approach scales with high traffic and how it interacts with caching mechanisms.
Exploring alternatives like using a token-based approach or leveraging existing frameworks or libraries.

#### Conclusion

By implementing an idempotent mechanism in FastAPI, we can ensure that requests to the create-order endpoint are processed safely even when they are retried. This involves generating unique keys, storing them temporarily, and checking for expiry before processing each request. Care must be taken to handle edge cases and scale appropriately.

**Comparison:**

```
Column	Type
key	VARCHAR
timestamp	DATETIME
```

**Code:**

```python
from fastapi import FastAPI, HTTPException, status
import uuid
from datetime import datetime, timedelta
from pydantic import BaseModel
from typing import Optional
from sqlalchemy.orm import Session
from your_database_setup import Base, IdempotencyKey  # Assuming a SQLAlchemy ORM setup

app = FastAPI()

class CreateOrderRequest(BaseModel):
    order_id: str
    product_name: str
    quantity: int

@app.post("/create-order")
async def create_order(request: CreateOrderRequest, db_session: Session):
    idempotency_key = uuid.uuid4()
    # Check if the key already exists and has not expired
    existing_key = db_session.query(IdempotencyKey).filter_by(key=idempotency_key).first()
    if existing_key and (datetime.utcnow() - existing_key.timestamp) < timedelta(minutes=5):
        raise HTTPException(status_code=status.HTTP_409_CONFLICT, detail="Request already processed")

    # Process the order
    db_session.add(IdempotencyKey(key=idempotency_key, timestamp=datetime.utcnow()))
    db_session.commit()

    # Simulate order creation
    result = {"order_id": request.order_id, "status": "created"}
    return result

# Example SQLAlchemy ORM setup for IdempotencyKey
class IdempotencyKey(Base):
    __tablename__ = "idempotency_keys"
    key = db.Column(db.String, primary_key=True)
    timestamp = db.Column(db.DateTime, default=datetime.utcnow)
```

**Example input / output:**

```text
Input:
POST /create-order HTTP/1.1
Host: example.com
Content-Type: application/json

{"order_id": "ord123", "product_name": "Widget", "quantity": 5}

Response:
HTTP/1.1 201 Created
Content-Type: application/json

{"order_id": "ord123", "status": "created"}
```

**Data:**

```sql
CREATE TABLE idempotency_keys (
    key VARCHAR(36) PRIMARY KEY,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Initiates Order
6. Check If Key Exists?
7. Return Conflicts
8. Generate Idempotency Key
9. Store Key and Timestamp inDB
10. Process Order
11. Simulate Order Creation
12. Check Expiry of Key?
13. Ignore Request
14. Return Success Response

