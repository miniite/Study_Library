---
type: "practice"
difficulty: "Deep"
---

# Design an authentication and session system for a web + mobile app — compare stateful sessions vs JWTs, and how do you handle logout/revocation and token refresh?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Micro-frontends](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Micro-frontends.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In designing an authentication and session system for a web and mobile application, we need to balance between stateful sessions and JSON Web Tokens (JWTs). Stateful sessions are managed on the server side with cookies or tokens stored in the database, while JWTs store all necessary information in the token itself. To handle logout/revocation and token refresh, we can use a combination of session invalidation techniques and token blacklisting.

#### Triage

The question asks us to compare stateful sessions and JWTs in terms of their suitability for an authentication system across both web and mobile platforms. Additionally, it seeks solutions on how to manage scenarios such as logout/revocation and token refresh effectively.

#### Assumptions
The application needs to support both web (browser-based) and mobile users.
There is a need to ensure the security of user sessions.
We are looking for a scalable solution that can handle a large number of concurrent users.

#### First-Principles Solution
1. Stateful Sessions

Stateful sessions involve maintaining session state on the server side, often using cookies or storing session IDs in a database.

Advantages:

Simplified implementation as state is managed by the backend.
Easier to integrate with existing systems that rely heavily on session data.

Disadvantages:

Limited scalability due to potential load balancing issues and session state synchronization across multiple servers.
Inconsistent behavior when users switch between devices or clear their cookies.
Requires database storage for session management, which can be a performance bottleneck.
2. JSON Web Tokens (JWTs)

JWTs are self-contained tokens that store user information encoded in the token itself. They do not require server-side state and can be managed client-side.

Advantages:

Stateless nature makes them highly scalable as they do not rely on backend state.
Easy to implement across different platforms, including mobile apps.
Can be securely transmitted over HTTP using HTTPS.

Disadvantages:

Requires careful handling of token expiration and revocation.
Potentially less secure if not managed properly (e.g., due to token leakage or weak secret keys).
3. Handling Logout/Revocation

To handle logout and revocation, we can implement a combination of techniques:

Token Revocation:

Use an access control list (ACL) mechanism where tokens are stored in a database with their expiration times.
Implement a token blacklist to keep track of revoked or invalid tokens.

Session Expiration and Token Refresh:

Implement a scheduled task that periodically checks for expired tokens and revokes them if necessary.
Provide a refresh endpoint that allows users to obtain new tokens using the old one, ensuring session continuity without requiring re-authentication.
4. Implementation Example

For a stateful session approach:

Yes
No
User Logs In
Server Generates Session ID
Client Stores Cookie withSession ID
Server Manages Session State
User Performs Actions
Client Sends Request withCookie
Server Checks Session State
Session Valid?
Access Granted
Redirect to Login

For a JWT approach:

Yes
No
User Logs In
Server Issues JWT with Claims
Client Stores JWT in LocalStorage
User Performs Actions
Client Sends Request with JWT
Server Verifies JWT
JWT Valid?
Access Granted
Redirect to Login, RevokeToken

#### Closing Summary

In summary, both stateful sessions and JWTs have their own trade-offs. Stateful sessions are easier to implement but less scalable, while JWTs offer better scalability at the cost of additional security considerations. To handle logout/revocation effectively, a combination of token revocation mechanisms and periodic session expiration checks can be employed.

#### Common Mistakes/Follow-Ups
Not Handling Token Revocation Properly: Failing to maintain a blacklist or properly invalidate tokens can lead to unauthorized access.
Over-Reliance on Stateless Tokens: While JWTs are stateless, they still require careful management of token lifetimes and revocation to prevent abuse.
Ignoring Edge Cases: Failure to account for scenarios like network issues or token expiration handling can lead to user frustration.

**Comparison:**

```
Token	Expiry Time
token123	2023-10-01T11:59:00Z
token456	2023-10-01T12:00:00Z
```

**Code:**

```python
# Example of a simple JWT revocation mechanism
from datetime import datetime, timedelta

def revoke_token(token):
    now = datetime.now()
    # Check if the token is in the blacklist
    if token in blacklist:
        return False
    # Expire the token after 30 minutes (for demonstration purposes)
    expiry_time = now + timedelta(minutes=30)
    if token['exp'] < expiry_time.timestamp():
        blacklist.add(token)
        return True
    return False

# Example of a session management mechanism
def manage_session(session_id):
    # Check the session state in the database
    if is_valid_session(session_id):
        return True
    else:
        # Mark the session as invalid and remove from the active sessions list
        invalidate_session(session_id)
        return False
```

**Example input / output:**

```text
Input: POST /token/refresh; Request Body: { "oldToken": "token123" }

Output:
{
  "status": "success",
  "newToken": "token789"
}
```

**Data:**

```json
{
  "blacklist": ["token123", "token456"],
  "sessions": {
    "session123": {"userId": 1, "expiryTime": "2023-10-01T12:00:00Z"}
  }
}
```

**Flow:**

1. Yes
2. No
3. User Requests Refresh
4. Verify Old Token
5. Token Valid?
6. Generate New Token
7. Deny Request

