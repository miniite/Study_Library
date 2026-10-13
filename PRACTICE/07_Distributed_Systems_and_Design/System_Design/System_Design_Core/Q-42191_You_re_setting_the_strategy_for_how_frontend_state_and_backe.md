---
type: "practice"
difficulty: "Expert"
---

# You're setting the strategy for how frontend state and backend cache stay coherent after a write: cache-aside invalidation, event-driven cache busting to the client, or server-push revalidation. Argue a default for a read-heavy social product and where you'd deviate.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Frontend Graceful Degradation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure coherence between frontend state and backend cache in a read-heavy social product like Instagram, we need to choose an effective strategy that minimizes data staleness while keeping performance high. The primary options are cache-aside invalidation, event-driven cache busting to the client, or server-push revalidation.

Cache-aside invalidation is straightforward: when the backend updates state, it doesn’t directly notify the frontend but instead marks the cache as stale and waits for the next request to fetch updated data. This approach ensures that the system remains responsive under high load since no additional communication overhead is incurred during a write operation.

Event-driven cache busting involves pushing notifications from the server to clients when relevant state changes occur, causing the client-side cache to be invalidated. This strategy requires careful consideration of real-time capabilities and can significantly impact server resources due to the constant stream of notifications.

Server-push revalidation combines elements of both methods by having the server push updated data directly to the client after a write operation. While this ensures immediate coherence, it comes at the cost of increased network traffic and could potentially overwhelm clients if not managed properly.

#### Triaging the Question

The core of the question is about balancing between minimal disruption during writes versus ensuring cache coherency on reads. The interviewer is likely probing your understanding of caching strategies, real-time updates, and the trade-offs involved in different approaches. Assumptions include:

The system has a significant read load.
The product requires quick access to updated data.
There are concerns about stale data affecting user experience.

#### First-Principles Solution

For a read-heavy social application like Instagram, cache coherency is crucial because users frequently need the most up-to-date information. Given this requirement and high read volume, cache-aside invalidation emerges as the default choice. Here’s why:

Low Write Overhead: Cache-aside invalidation minimizes the impact of writes on the server by not requiring immediate client notifications or revalidations.
Responsiveness Under High Load: The system can handle large numbers of concurrent reads without significant delays, ensuring a smooth user experience even during peak usage.
Efficient Use of Resources: By only invalidating caches at the next read request, resources are conserved on both the server and client sides.

However, there may be scenarios where deviating from this default is justified:

High User Interaction: If the application frequently involves user interactions that require immediate updates (e.g., notifications), an event-driven approach might be beneficial.
Real-Time Updates: For features requiring real-time data synchronization between clients and servers, such as live chat or collaborative editing tools, server-push revalidation could provide a better experience.
Resource Constraints on Clients: In environments where client resources are limited (e.g., mobile devices), event-driven cache busting might be necessary to manage battery usage.

#### Closing Summary

In summary, for a read-heavy social product like Instagram, cache-aside invalidation is the default strategy due to its efficiency and low overhead during writes. However, specific scenarios may necessitate deviations based on user interaction patterns or resource constraints.

#### Common Mistakes
and Follow-Ups:

Failing to Consider Real-Time Requirements: Assuming all applications can operate solely with batched updates.
Ignoring Client Resources: Not accounting for the impact of frequent notifications on clients, especially on mobile devices.
Overlooking Granularity: Treating all write operations equally without considering their importance or frequency.

In follow-up discussions, you might explore more nuanced approaches such as hybrid strategies that combine elements of different methods based on specific use cases. For instance, implementing cache-aside invalidation for core functionalities and event-driven updates for real-time interactions.

**Comparison:**

```
Strategy	Write Overhead	Read Performance	Real-Time Capabilities
Cache-Aside Invalidation	Low	High	None
Event-Driven Cache Busting	Medium to High	High	Yes
Server-Push Revalidation	High	Very High	Yes
```

**Code:**

```python
# Example code snippet for a simple cache invalidation mechanism
def update_user_cache(user_id):
    # Backend logic to update user data
    updated_data = fetch_updated_user_data(user_id)

    # Mark the cache as stale without directly notifying clients
    mark_cache_as_stale(user_id)

    return updated_data

# Example of event-driven cache busting
def push_notification(user_id, event_type):
    # Send a notification to the client
    send_client_notification(user_id, event_type)

    # Invalidate local cache on the client side if necessary
    invalidate_local_cache(user_id)
```

**Example input / output:**

```text
Input: User Alice updates her profile picture.
Output: Backend processes the update and marks Cache-aside as stale for Alice's cache. Next read request from any client fetches updated data.

Input: Server sends a notification to Client Bob about a new message in his chat with Alice.
Output: Client Bob receives the push notification, invalidates local cache, and fetches latest messages.
```

**Data:**

```json
{
  "user_id": 123,
  "event_type": "update",
  "timestamp": "2023-10-01T14:00:00Z"
}
```

**Flow:**

1. Client
2. Backend
3. No
4. Yes
5. User Alice updates profile
6. Is cache invalidated?
7. Backend processes update,marks Cache-aside as stale
8. Next read from Client fetchesupdated data
9. Client receives pushnotification, invalidates localcache
10. Client fetches latest data

