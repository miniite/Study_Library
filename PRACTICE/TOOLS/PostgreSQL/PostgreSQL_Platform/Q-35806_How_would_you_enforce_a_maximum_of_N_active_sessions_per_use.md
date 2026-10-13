---
type: "practice"
difficulty: "Deep"
---

# How would you enforce a maximum of N active sessions per user at the database level to prevent runaway concurrency, and what are the pitfalls of enforcing this with a trigger versus in the app?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Row-level Security](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a maximum of (N) active sessions per user at the PostgreSQL database level, we need to manage session state effectively to prevent runaway concurrency issues. This can be achieved either by using a trigger in the database or implementing logic in the application layer. However, each approach has its own set of trade-offs and pitfalls that must be carefully considered.

Enforcing this restriction directly in the database is more efficient because it reduces the load on the application server and ensures that session limits are consistently enforced regardless of how many users are interacting with the system simultaneously. The key idea here is to maintain a dedicated table or index that tracks active sessions per user, updating these records as new sessions begin and ending them when they terminate.

#### Triage

The core issue at hand is managing concurrent access effectively by limiting the number of active sessions per user. This can be done using database-level constraints or application logic. The interviewer might be probing your understanding of concurrency control mechanisms, database triggers, and the implications of state management in distributed systems.

#### First-Principles Solution
Using a Database-Level Approach with Triggers

Create a Sessions Table:

SQL
CREATE TABLE user_sessions (
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users (id),
    session_start TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    session_end TIMESTAMP NULL
);

Trigger to Enforce Session Limits: We can create a trigger that ensures no more than (N) active sessions exist for any given user. This trigger will block new sessions if the limit is exceeded.

Trigger Implementation Example:

SQL
CREATE OR REPLACE FUNCTION enforce_session_limit() RETURNS TRIGGER AS $$
BEGIN
    -- Count current active sessions for this user
    IF (SELECT COUNT(*) FROM user_sessions WHERE user_id = NEW.user_id AND session_end IS NULL) < N THEN
        RETURN NEW;
    ELSE
        RAISE EXCEPTION 'User % has reached the maximum number of sessions (%).', NEW.user_id, N;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER enforce_session_limit_trigger BEFORE INSERT ON user_sessions FOR EACH ROW EXECUTE FUNCTION enforce_session_limit();

Handling Session Termination: It's crucial to update the session_end timestamp correctly when a session terminates, which can be done using another trigger or application logic.

Using Application Logic

Maintain User Sessions State in App Layer:

The application keeps track of active sessions per user.
A dedicated service layer manages this state and enforces the limit before allowing new sessions.

Session Management API:

PYTHON
from threading import Lock

class SessionManager:
    def __init__(self, max_sessions_per_user):
        self.lock = Lock()
        self.sessions_per_user = {}

    def start_session(self, user_id):
        with self.lock:
            if len(self.sessions_per_user.get(user_id, [])) >= max_sessions_per_user:
                raise Exception("User has reached the maximum number of sessions.")
            self.sessions_per_user.setdefault(user_id, []).append(None)
            return True

    def end_session(self, session_id):
        with self.lock:
            for i, _ in enumerate(self.sessions_per_user.get(session_id, [])):
                if self.sessions_per_user[session_id][i] is None:
                    self.sessions_per_user[session_id][i] = datetime.now()
                    break

#### Closing Summary
+ Common Mistakes/Follow-Ups
Database Approach: Efficient but requires careful trigger logic and might introduce complexity in schema management.
Application Logic: Simpler to implement but may add load on the application server if many users are interacting simultaneously.

Common pitfalls include:

Race Conditions: Ensure that session state updates are atomic, especially when multiple sessions can be created or terminated concurrently.
Scalability: Consider how the database and application will scale under heavy load.
Maintenance: Triggers require maintenance and tuning to ensure they perform well as the system grows.

For follow-ups, you might discuss alternative approaches like using distributed locks or external state managers (like Redis) to handle session limits more robustly.

**Comparison:**

```
has
USER
SESSION
int
id
uuid
user_id
timestamp
session_start
timestamp
session_end
```

**Code:**

```sql
CREATE TABLE user_sessions (
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users (id),
    session_start TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    session_end TIMESTAMP NULL
);

CREATE OR REPLACE FUNCTION enforce_session_limit() RETURNS TRIGGER AS $$
BEGIN
    IF (SELECT COUNT(*) FROM user_sessions WHERE user_id = NEW.user_id AND session_end IS NULL) < N THEN
        RETURN NEW;
    ELSE
        RAISE EXCEPTION 'User % has reached the maximum number of sessions (%).', NEW.user_id, N;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER enforce_session_limit_trigger BEFORE INSERT ON user_sessions FOR EACH ROW EXECUTE FUNCTION enforce_session_limit();
```

**Example input / output:**

```text
-- Insert a new session that would be blocked if the limit is reached
INSERT INTO user_sessions (user_id) VALUES ('12345678-90ab-cdef-fedc-ba9876543210');

-- This will raise an exception because the maximum number of sessions has been reached
```

**Data:**

```json
{
  "user_sessions": [
    {
      "id": 1,
      "user_id": "12345678-90ab-cdef-fedc-ba9876543210",
      "session_start": "2023-10-01T12:00:00Z",
      "session_end": null
    },
    {
      "id": 2,
      "user_id": "12345678-90ab-cdef-fedc-ba9876543210",
      "session_start": "2023-10-01T12:30:00Z",
      "session_end": null
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Starts a New Session
4. Check Number of ActiveSessions
5. Is the Limit Reached?
6. Raise Exception and BlockSession
7. Allow Session to Begin

