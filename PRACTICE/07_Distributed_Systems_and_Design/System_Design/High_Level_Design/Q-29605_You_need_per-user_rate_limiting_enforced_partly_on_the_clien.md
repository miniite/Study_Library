---
type: "practice"
difficulty: "Intermediate"
---

# You need per-user rate limiting enforced partly on the client (to give instant UI feedback) and authoritatively on the server — how do you split and reconcile the two?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiter Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively implement per-user rate limiting, we need to ensure that both the client and the server enforce limits in a way that provides consistent user experience and accurate tracking of API calls. The primary challenge is to split responsibilities between the client and server while maintaining consistency and reliability.

#### Triaging the Question

The question asks us to design a solution where:

Client-Side Enforcement: Provide immediate UI feedback as soon as the rate limit is hit.
Server-Side Authority: Ensure that all rate limits are ultimately enforced on the server, providing authoritative and final decision-making.

#### Assumptions
and Key Considerations
The client has limited resources compared to the server (e.g., slower updates).
Network latency can introduce inconsistencies between the client and server states.
We need to handle edge cases like network failures or user actions that occur during transitions between client and server updates.

#### Cons
istency should be prioritized over performance in critical situations.

#### Solution
from First Principles
Client-Side Initialization: The client will maintain a local state of each user's rate limit and usage count, providing immediate feedback to the UI.
Server-Side Validation: Upon every API call, the client will send the current rate limit usage to the server for validation before making the actual request.
Rate Limit State Synchronization: Periodically or upon critical state changes (e.g., server enforcing limits), the client and server will synchronize their states.
Step-by-Step Solution

Client Initialization:

The client initializes with a rate limit value from the server.
It maintains a local counter for each API call type per user.

Client-Side Rate Limit Check:

Before sending an API request, the client checks its local state to see if the current action would exceed the allowed usage.
If exceeded, it immediately informs the UI (e.g., show a "Rate Limit Exceeded" message).
It also sends the updated count and remaining limit to the server for validation.

Server-Side Validation:

Upon receiving the client's request, the server checks its own state against the received data.
If the validation passes, it allows the request; otherwise, it rejects the request and returns an appropriate error message.
The server updates its internal state to reflect any necessary changes.

State Synchronization:

Periodically or upon critical events (e.g., a user exceeding their limit), both client and server will synchronize their states.
This ensures that any discrepancies are resolved, maintaining consistency.

#### Implementation
Example

Here’s an example flow for a typical API call:

PLAINTEXT
Client State: { userId: 1, callsPerMin: 20, remainingCalls: 18 }
User Action: Make API Call #19

// Client-side check fails (remainingCalls == 0)
UI Message: "Rate Limit Exceeded"
Client sends updated state to Server: { userId: 1, remainingCalls: 17 }

Server receives the request and:
- Checks its own state
- Updates state if necessary
- Allows or rejects based on new state

// Client receives confirmation from server and updates local state accordingly

#### Closing Summary
+ Common Mistakes/Follow-ups

To effectively implement per-user rate limiting, it’s crucial to ensure that both the client and server play their roles correctly while maintaining consistency. Key takeaways include:

Use a hybrid approach combining client-side feedback with authoritative server validation.
Handle edge cases where network latency or other disruptions might cause inconsistencies.

#### Common mistakes
to avoid:

Ignoring Server Validation: Relying solely on client-side enforcement can lead to inconsistent experiences and potential security vulnerabilities.
Failing to Synchronize States: Without proper state synchronization, discrepancies can arise, leading to inaccurate rate limiting enforcement.

#### Follow-ups
could include discussions about different techniques for reducing network latency (e.g., WebSockets) or implementing more sophisticated state management strategies.

**Comparison:**

```
API Call	Client Remaining Calls	Server Remaining Calls
Initial State	18	18
After Call #19 (Client)	0	17
After Server Validation	N/A	17
```

**Code:**

```

// Example client-side code snippet class RateLimiter { constructor(userId, maxCallsPerMinute) { this.userId = userId; this.maxCallsPerMinute = maxCallsPerMinute; this.callsMadeThisMinute = 0; }

makeAPICall() { if (this.callsMadeThisMinute >= this.maxCallsPerMinute) { console.log("Rate Limit Exceeded"); // UI update logic here return false; // Block the API call }

CODE
Copy
this.callsMadeThisMinute++;
// Send updated state to server for validation

} }

CODE
Copy

```javascript
// Example server-side code snippet
class RateLimiterService {
  constructor(userId, maxCallsPerMinute) {
    this.userId = userId;
    this.maxCallsPerMinute = maxCallsPerMinute;
    this.callsMadeThisMinute = 0;
  }

  validateCall(data) {
    if (data.remainingCalls <= 0) {
      return false; // Reject the request
    }

    data.remainingCalls--;
    // Update server state accordingly

    return true; // Allow the request
  }
}
```

**Example input / output:**

```text
Client Request:
- userId: 1
- remainingCalls: 17

Server Response:
- validationResult: true/false
- updatedRemainingCalls: integer

Sync Synchronization Example:
- Client sends state to Server
- Server updates its state and sends back confirmation
```

**Data:**

```json
{
  "userId": 1,
  "maxCallsPerMinute": 20,
  "remainingCalls": 18
}
```

**Flow:**

1. No
2. Client Initiates API Call
3. Check Local Rate Limit
4. Is Remaining Calls <= 0?
5. Send Updated State to Serverfor Validation
6. Server Validates and UpdatesState
7. Notify Client of Result

