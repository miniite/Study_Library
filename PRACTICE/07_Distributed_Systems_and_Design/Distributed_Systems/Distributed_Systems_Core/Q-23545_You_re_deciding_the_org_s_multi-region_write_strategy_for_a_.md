---
type: "practice"
difficulty: "Expert"
---

# You're deciding the org's multi-region write strategy for a system where users are geo-pinned but occasionally roam. Compare home-region routing with conflict handling on move, CRDT-based multi-master, and a global consensus group, and explain how you'd handle a user who moves mid-session.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quorums and Consensus](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Quorums_and_Consensus.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding a multi-region write strategy for a geo-pinned system with occasional roaming users, we need to balance consistency, availability, and partition tolerance (CAP theorem). The key options are home-region routing with conflict handling on move, CRDT-based multi-master, and global consensus. Each approach has its trade-offs in terms of complexity, performance, and data integrity.

Home-region routing involves directing writes to the user's designated region while handling conflicts locally or centrally when the user roams. This method is simple but can lead to stale data if a conflict is not resolved quickly. CRDT-based multi-master allows concurrent updates across regions, ensuring eventual consistency. However, this introduces complex conflict resolution logic and potential data divergence issues. Global consensus ensures strong consistency by coordinating writes through a central group. This approach guarantees correctness at the cost of lower availability during network partitions.

To handle a user who moves mid-session, we must consider how these strategies manage the transition from one region to another.

**Comparison:**

```
Strategy	Consistency	Availability	Partition Tolerance	Complexity
Home-region routing	Weak	High	Medium	Low
CRDT-based multi-master	Eventual	Medium	High	High
Global consensus	Strong	Low	High	High
```

**Code:**

```python
# Example code for home-region routing with conflict handling on move
class User:
    def __init__(self, home_region):
        self.home_region = home_region
        self.last_known_region = None

    def write(self, region, data):
        if region == self.home_region or region == self.last_known_region:
            # Handle local write
            pass
        else:
            # Handle conflict on move
            if not handle_conflict_on_move(region):
                raise ValueError("Conflict detected during user movement")
```

**Example input / output:**

```text
User with ID 1 (originally in us-west-2) starts writing data from us-east-1.
Initial state:
{
  "users": [
    {"userId": 1, "homeRegion": "us-west-2", "lastKnownRegion": null},
    {"userId": 2, "homeRegion": "eu-central-1", "lastKnownRegion": "eu-central-1"}
  ]
}

Input: Write request from user ID 1 in us-east-1.
Output: Conflict detected during user movement. User state:
{
  "users": [
    {"userId": 1, "homeRegion": "us-west-2", "lastKnownRegion": "us-east-1"},
    {"userId": 2, "homeRegion": "eu-central-1", "lastKnownRegion": "eu-central-1"}
  ]
}
```

**Data:**

```json
{
  "users": [
    {"userId": 1, "homeRegion": "us-west-2", "lastKnownRegion": null},
    {"userId": 2, "homeRegion": "eu-central-1", "lastKnownRegion": "eu-central-1"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. User moves from us-west-2 tous-east-1
4. Conflict detected?
5. Handle conflict locally orcentrally
6. Write to home region(us-west-2)

