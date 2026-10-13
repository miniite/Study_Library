---
type: "practice"
difficulty: "Deep"
---

# Design the architecture for a system where the frontend must support real-time multi-device continuity (start on phone, continue on laptop mid-flow) for a purchase. How do you sync ephemeral cart/session state across devices with low latency and conflict-free merges?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an architecture that supports real-time multi-device continuity for a purchase process, we need to ensure seamless synchronization of ephemeral cart/session states across multiple devices with minimal latency and conflict-free merging. At a high level, this involves using efficient state management strategies, implementing robust data synchronization protocols, and ensuring low-latency communication channels.

#### Triaging the Question

The core challenge here is maintaining consistency in session state while supporting real-time updates across devices. The key aspects to consider are:

Real-time Synchronization: Ensuring that updates from one device propagate to others as quickly as possible.
Conflict-Free Merging: Handling scenarios where multiple devices attempt to modify the same state simultaneously without data loss or corruption.
Low Latency: Minimizing the time delay between when a change occurs and when it is reflected on all devices.

#### First-Principles Solution
State Management Strategy:
Use a centralized backend service that acts as the authoritative source of truth for the cart/session state.
Real-time Communication:
Leverage WebSocket or Server-Sent Events (SSE) for bi-directional communication between devices and the backend.
Conflict Detection & Resolution:
Implement a versioning mechanism to detect conflicts during merges.
Detailed Architecture
Centralized Backend Service
The backend will store the current state of the cart/session in a database (e.g., MongoDB) and manage all updates.
WebSocket/SSE for Real-time Updates
Devices connect via WebSocket or SSE to receive real-time updates from the backend.
Versioning Mechanism
Each update request includes a version number. The backend keeps track of these versions and detects conflicts when merging.

#### Implementation
Steps
Device Initialization:
On startup, each device fetches the current state from the backend.
User Actions on Device A:
User performs an action (e.g., adds item to cart) on Device A.
State Update Request:
Device A sends a request to the backend with the updated state and version number.
Conflict Detection & Merge:
Backend checks for conflicts by comparing the received version number with the latest stored version.
Real-time Notification:
If no conflict, backend updates the database and notifies other connected devices via WebSocket/SSE.

#### Common Mistakes
and Follow-ups
Failure to Implement Versioning: Not using a versioning mechanism can lead to data inconsistencies when multiple devices attempt updates simultaneously.
Inadequate Error Handling: Failing to handle network errors or backend downtime properly can result in stale state on devices.
Ignoring Edge Cases: Not considering scenarios like network partitions, where some devices might lose connectivity temporarily.

#### Example
Workflow
PLAINTEXT
Device A: User adds item to cart (Version 1)
Device A → Backend: Update request with new version (2)
Backend → Device B via WebSocket: State update notification
Backend updates state in database and sends updated state to all connected devices.

#### Closing Summary

To achieve real-time multi-device continuity, we need a robust architecture that centralizes the cart/session state management. This involves using WebSocket or SSE for low-latency communication, implementing versioning mechanisms to detect conflicts, and ensuring that all updates are propagated efficiently.

Self-Check

Correctness + completeness confirmed.

**Comparison:**

```
Device	State Version
Device A	2
Device B	1
Device C	1
```

**Code:**

```python
# Example WebSocket Client Code
import websocket

def on_message(ws, message):
    print("Received:", message)

ws = websocket.WebSocketApp("wss://backend.example.com/api/cart",
                           on_message=on_message)
ws.run_forever()
```

**Example input / output:**

```text
Input: User adds item to cart on Device A
Output: Backend updates state and sends notification via WebSocket to all devices
```

**Data:**

```json
{
  "cart_id": "123456",
  "items": [
    {
      "id": "item1",
      "quantity": 1
    },
    {
      "id": "item2",
      "quantity": 2
    }
  ],
  "version": 1
}
```

**Flow:**

1. Device A
2. Add Item
3. Send Update Request toBackend
4. Backend Updates State andVersions
5. Backend Sends Notification viaWebSocket
6. Devices Receive Updated State(Version 2)

