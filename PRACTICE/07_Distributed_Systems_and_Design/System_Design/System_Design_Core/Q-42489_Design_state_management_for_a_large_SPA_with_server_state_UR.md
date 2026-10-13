---
type: "practice"
difficulty: "Intermediate"
---

# Design state management for a large SPA with server state, URL state, and ephemeral UI state. When would you reach for a global store vs local state vs server-cache, and what's over-engineering here?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Frontend Graceful Degradation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design state management for a large Single-Page Application (SPA) that integrates with both server-side and client-side states, we need to consider multiple factors including performance, scalability, consistency, and complexity. The core challenge is managing three types of state: global server state, URL state, and ephemeral UI state. This requires balancing between using a global store, local state, or leveraging the power of server caching.

#### Triage
The question is asking how to choose among different state management strategies based on the context—whether it's appropriate to use a global store, local state, or server cache. The interviewer is probing your understanding of state complexity and your ability to make trade-offs between performance, consistency, and development simplicity.

Key Assumptions:

We are designing an SPA that integrates with backend services.
Different states have varying lifetimes and update frequencies.
Performance and user experience are critical considerations.

#### First-principles solution

Global Server State: Use when data must be consistent across all instances of the application, such as session management or global configurations. This can involve using REST APIs or WebSocket connections to fetch and push state updates to clients.

URL State: Manage using URL parameters or hash fragments for simpler state changes that don't require real-time updates, like sorting options in a list view. For more complex scenarios, consider using query parameters or storing state in local storage/session storage with client-side routing libraries.

Ephemeral UI State: Use local state (e.g., React/Vue's useState/reactive) for short-lived, client-specific data that doesn't need to be shared between users or cached on the server. This includes user interactions and temporary UI updates.

When deciding between a global store and local state:

Global Store is ideal for managing complex states that require consistency across multiple components and persistency.
Local State is better when you want to keep data within a single component's scope, reduce network calls, and improve performance.

Server caching can be over-engineered if the data doesn't frequently change or if real-time updates are not critical. It introduces additional complexity in managing cache invalidations and increasing latency for reads during write operations.

#### Closing Summary
+ Common Mistakes/Follow-Ups: The key is understanding that different state types have distinct requirements, and choosing the right approach depends on the specific context. Over-engineering can introduce unnecessary complexity and reduce performance. Always consider the trade-offs between consistency, performance, and development simplicity.

#### Common mistakes
include:

Using a global store for every piece of data, leading to excessive network requests.
Not considering real-time updates when they are not necessary.
Ignoring local state optimization by always opting for global solutions.

#### Follow-ups
might explore how you would implement a specific feature using these strategies or how you would balance between different states in a real-world application.

**Comparison:**

```
State Type	Example Use Case	When to Use
Global	User authentication	For session management
URL	Sorting options	Simple UI interactions
Ephemeral	Local interaction state	Client-specific data
```

**Code:**

```javascript
// Example of local state management with React hooks
function MyComponent() {
  const [state, setState] = useState(initialState);

  useEffect(() => {
    // Update UI based on URL changes
    window.addEventListener('hashchange', () => {
      // Fetch new data from server if needed
      fetchNewData();
    });
    return () => {
      window.removeEventListener('hashchange');
    };
  }, []);

  function handleUserInteraction() {
    setState(newState);
    // Local interaction handling without affecting global state
  }

  return (
    <div>
      {/* Render UI based on current state */}
    </div>
  );
}
```

**Example input / output:**

```text
Input:
- Global server state update request

Output:
1. Server receives the request and updates the relevant data.
2. Client fetches updated data via API call or WebSocket event.
3. Local state components in the SPA are updated based on new global state.

Request->Response Flow:
- Client sends a POST /update-global-state request with new data.
- Server processes the request, updating database entries.
- Server responds with 200 OK and new state data.
- Client updates local state using this data.
```

**Data:**

```json
{
  "state": {
    "globalServerState": true,
    "urlState": "/users?sort=desc",
    "uiState": { "selectedTab": "profile" }
  },
  "actions": [
    { "type": "UPDATE_GLOBAL", "data": {} },
    { "type": "CHANGE_URL", "data": { "newUrl": "/settings" } },
    { "type": "TOGGLE_UI", "data": { "tab": "dashboard" } }
  ]
}
```

**Flow:**

1. No
2. Yes
3. User Interaction
4. Update Local State
5. Check if Real-Time UpdateNeeded
6. No Action
7. Fetch Data from Server
8. Update UI Based on New Data
9. URL Change
10. Update URL Parameters/Hash

