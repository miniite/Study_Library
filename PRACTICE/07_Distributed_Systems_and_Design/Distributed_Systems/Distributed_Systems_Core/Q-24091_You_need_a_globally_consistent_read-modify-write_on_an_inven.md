---
type: "practice"
difficulty: "Deep"
---

# You need a globally consistent read-modify-write on an inventory row that must never oversell, across a distributed store. Compare an optimistic version-column CAS loop against SELECT FOR UPDATE with a distributed lock on latency and abort rate.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of ensuring globally consistent read-modify-write operations without overselling inventory in a distributed system, we need to compare two approaches: an optimistic version-column CAS (Compare-and-Swap) loop and a pessimistic approach using SELECT FOR UPDATE with distributed locks. The goal is to understand their trade-offs in terms of latency and abort rate.

**Comparison:**

```
Metric	CAS Loop	SELECT FOR UPDATE
Latency	Lower, as it avoids locks	Higher due to contention
Abort Rate	Higher, due to retries	Lower, but potential deadlocks
Scalability	Good, with concurrent reads	Limited by lock contention
```

**Code:**

```
Example code for CAS loop

def read_modify_write_cas(row_id, expected_version): while True: current_row = get_current_row(row_id) new_version = current_row['version'] + 1 if current_row['version'] == expected_version: if decrement_inventory(current_row['quantity']): set_current_row(row_id, {'version': new_version}) return True else: # version has changed; retry pass

def get_current_row(row_id): # Get the latest row from the distributed store pass

def decrement_inventory(quantity): # Decrement the inventory quantity and check if we can proceed pass

CODE
Copy

```python
# Example code for SELECT FOR UPDATE with distributed lock
from contextlib import contextmanager

@contextmanager
def acquire_distributed_lock(lock_key):
    try:
        # Acquire a distributed lock on the lock_key
        acquire_lock(lock_key)
        yield
    finally:
        release_lock(lock_key)

def read_modify_write_lock(row_id):
    with acquire_distributed_lock(f'inventory:{row_id}'):
        current_row = get_current_row(row_id)
        if decrement_inventory(current_row['quantity']):
            set_current_row(row_id, {'version': current_row['version'] + 1})
```

**Example input / output:**

```

Input:

row_id: 123

CAS Loop Output:

CODE
Copy
# First attempt to update row with version = 5
read_modify_write_cas(123, 5)
# Update is successful; row updated with new version 6

SELECT FOR UPDATE Output:

CODE
Copy
with acquire_distributed_lock('inventory:123'):
    current_row = get_current_row(123)
    if decrement_inventory(current_row['quantity']):
        set_current_row(123, {'version': current_row['version'] + 1})
# Update is successful; row updated with new version 6
```

**Data:**

```json
{
    "row_id": 123,
    "version": 5,
    "quantity": 100
}
```

**Flow:**

1. Read current row and getversion
2. CAS Loop: Compare-and-Swap
3. Avoids locks; Lower latency
4. Increment retry count ifversion mismatch
5. Update successful; Exit loop
6. Abort if retries exceed limit
7. Read current row and getversion
8. SELECT FOR UPDATE: Acquirelock
9. Decrement inventory
10. Lock held; Update successful;Release lock
11. Abort if cannot acquire lock orupdate fails

