---
type: "practice"
difficulty: "Expert"
---

# You expose GET /accounts/:id/balance and it must reflect writes made microseconds earlier from the same client but tolerate replica lag for everyone else. How do you route reads to satisfy both without a strong-read for all?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consistent Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to ensure that our GET /accounts/:id/balance endpoint reflects recent writes made by the same client while tolerating replica lag for other clients. This requires a nuanced approach involving both consistency and availability trade-offs. Let’s break down the solution step-by-step.

Triaging the Problem

The core of the question is how to route reads in such a way that they reflect the latest state seen by the writing client, but also handle eventual consistency gracefully for other clients. This involves understanding the nature of read-write consistency models and leveraging caching mechanisms.

#### Assumptions
We are using a distributed system with multiple replicas.
The writes are timestamped or version-controlled to determine freshness.
There is a mechanism to detect if a client has recently written to an account (id).
Caches have some expiration time or invalidation mechanism.

#### First Principles Solution
Client-Side Tracking: When the client makes a write, it should record its own timestamp and correlate this with future reads.
Read Routing Logic:
For reads initiated by the writing client: Route directly to the primary (or the most recent replica).
For all other clients: Use a distributed cache or read-replica mechanism that reflects the latest state, while still allowing for some degree of stale data.
Step-by-Step Implementation

Client Write Mechanism:

The client sends a write request to the system.
Upon successful write, the client records the timestamp and any relevant session information (e.g., token ID) in its local state.

Read Handling for Writing Client:

The client includes the recorded timestamp/session identifier with the GET request.
If a read request matches this timestamp/session identifier, route it directly to the primary replica or most recent replica.

Read Handling for Other Clients:

For all other clients, use a distributed cache (e.g., Redis) that holds the latest account balance information.
Cache invalidation can be done via TTLs or manual invalidations based on write operations.
Detailed Example

Assume we are using a read-write separation pattern with Redis as our cache. Here's how it might look in practice:

PYTHON
# Client-side logic (simplified pseudo-code)
def make_write_request(account_id, balance):
    timestamp = current_timestamp()
    # Write request to database with timestamp
    db.write(account_id, balance, timestamp)

    # Record the write for future reads
    session_state = {
        "account_id": account_id,
        "timestamp": timestamp,
        "balance": balance
    }

def make_read_request(account_id):
    if client_session_data and is_recent_write(client_session_data['account_id'], client_session_data['timestamp']):
        return direct_read_from_primary(account_id)
    else:
        return read_from_cache(account_id)

# Server-side logic (simplified pseudo-code)
@app.route('/accounts/:id/balance', methods=['GET'])
def get_balance(id):
    if is_write_request_made_recently(id):
        # Route to primary or most recent replica
        return direct_read(id)
    else:
        # Use cache (Redis) with TTL or manual invalidation
        return read_from_cache(id)

# Cache Invalidation Logic
@app.route('/accounts/:id/write', methods=['POST'])
def write_balance(id, balance):
    timestamp = request.headers.get('X-Timestamp')
    db.write(account_id, balance, timestamp)

    # Invalidate cache for this account id
    cache_invalidate(id)

#### Closing Summary
and Common Mistakes

In summary, the key is to use client-side tracking combined with a distributed caching mechanism. This approach ensures that the writing client sees its own writes immediately while others see eventual consistency.

#### Common Mistakes

Failing to track timestamps or session identifiers accurately.
Not leveraging cache invalidation properly, leading to stale reads for other clients.
Over-relying on a single primary replica, which can cause availability issues if it fails.

By carefully designing the read routing logic and implementing robust caching mechanisms, we can achieve the desired consistency levels without compromising too much on availability.

