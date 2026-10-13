---
type: "practice"
difficulty: "Deep"
---

# Your read replica lags and users hit read-your-writes bugs after a POST; design a class-level 'sticky-to-primary' or version-token approach so a user reads their own write consistently without forcing all reads to primary.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Maintaining Denormalized Views](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the issue of users hitting read-your-writes bugs after a POST, especially when there is a lag in the read replica, we can design a class-level 'sticky-to-primary' or version-token approach. This method ensures that each user's reads are always consistent with their own writes by tracking and validating write operations against a primary node. Here’s how it works:

#### Triage
What the Question Is Really Asking

The question asks for an architectural solution to manage read consistency in the presence of lag between the primary database and its replicas. Specifically, we need to ensure that after a user performs a write operation (POST), their subsequent reads will always reflect this write, even if the replica is lagging.

#### Assumptions
There are multiple users interacting with the system.
Users typically read data from both primary and replicas for performance reasons but prefer writes to be atomic on the primary.
Replicas may lag behind the primary by a few milliseconds to seconds.
The application needs to maintain high availability and low latency.

#### First-Principles Solution
Token Generation: Each time a user performs a write operation, generate a unique token (version number) that is associated with this specific write operation.
Sticky Session Management: Use sticky session management techniques to ensure that the user's subsequent read operations are directed back to the same node where their last write was performed.
Validation on Read Operations: Before performing any read operations, validate the token against the primary database to ensure that the read is consistent with the latest state of the data.
Detailed Steps:
Write Operation:
User initiates a POST request to create or update an entity.
The application generates a unique version token for this write operation.
The write operation is logged in both the primary and replica databases, but only with the same version token.
Read Operation:
When a user needs to read data, their session is tracked so that they are always directed back to the node where their last write was performed (sticky session management).
The system retrieves the latest version token from this node and validates it against the primary database before performing any reads.

#### Cons
istency Check:
If the local version token matches the version in the primary, the read is allowed.
If there's a mismatch, the request is redirected to the primary for validation or resynchronized.

#### Closing Summary
+ Common Mistakes/Follow-ups

This approach ensures that reads are always consistent with the user’s latest writes by leveraging sticky sessions and version tokens. It minimizes the impact of replica lag while maintaining data integrity. Key challenges include managing session stickiness, ensuring efficient validation mechanisms, and handling edge cases like concurrent writes or network partitions.

#### Common mistakes
might involve not properly tracking versions or failing to implement effective sticky session management, leading to inconsistent reads for users. Follow-ups could include discussing how to handle write conflicts during the validation process or exploring alternative approaches such as conflict resolution strategies in distributed systems.

**Comparison:**

| Step | Action                | Primary Database  | Replica Database |
|------|-----------------------|-------------------|------------------|
| 1    | User initiates write  | Write with token  | Write with token |
| 2    | Validate read         | Check version     | Read from replica|
| 3    | Handle mismatch       | Redirect to primary|

**Code:**

```python
class StickyToPrimary:
    def __init__(self):
        self.session_tokens = {}

    def handle_write(self, user_id, data):
        # Generate a unique version token
        version_token = self.generate_version_token()

        # Perform write operation on both primary and replica with the same version token
        self.write_to_primary(user_id, data, version_token)
        self.write_to_replica(user_id, data, version_token)

        return version_token

    def handle_read(self, user_id):
        session_token = self.session_tokens.get(user_id)
        if not session_token:
            # Handle case where no token is found (e.g., initiate a write first)
            raise Exception("No session token available")

        latest_version_in_primary = self.get_latest_version_from_primary()
        if session_token == latest_version_in_primary:
            return self.read_from_replica(user_id)  # Read from replica if token matches
        else:
            return self.read_from_primary(user_id)  # Read directly from primary for validation

    def write_to_primary(self, user_id, data, version_token):
        # Write to primary database with the version token
        pass

    def write_to_replica(self, user_id, data, version_token):
        # Write to replica database with the version token
        pass

    def read_from_replica(self, user_id):
        # Read from replica and return data
        pass

    def read_from_primary(self, user_id):
        # Read directly from primary for validation
        pass

    def generate_version_token(self):
        # Generate a unique version token (e.g., UUID)
        return "version_token_" + str(uuid.uuid4())

    def get_latest_version_from_primary(self):
        # Retrieve the latest version token from the primary database
        pass

# Example usage
sticky_to_primary = StickyToPrimary()
version_token = sticky_to_primary.handle_write("user1", {"key": "value"})
sticky_to_primary.handle_read("user1")
```

**Example input / output:**

```text
Write Operation:
Input: POST /write?user=user1&data={"key": "value"}
Output: {"status": "success", "token": "version_token_001"}

Read Operation (Sticky Session):
Input: GET /read?user=user1
Output: {"status": "success", "data": {"key": "value"}}
```

**Data:**

```json
{
  "write_request": {
    "user_id": "user1",
    "data": {"key": "value"},
    "version_token": "version_token_001"
  },
  "read_request": {
    "user_id": "user1",
    "version_token": "version_token_001"
  }
}
```

**Flow:**

1. Yes
2. No
3. User Initiates Write
4. Primary Logs Write
5. Replica Logs Write
6. Validate Version Token?
7. User Reads from Replica
8. Read Directly from Primary

