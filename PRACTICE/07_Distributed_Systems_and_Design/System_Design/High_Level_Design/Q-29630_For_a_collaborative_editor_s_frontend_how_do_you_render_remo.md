---
type: "practice"
difficulty: "Deep"
---

# For a collaborative editor's frontend, how do you render remote cursors, apply incoming ops smoothly, and keep the local doc responsive while syncing over WebSocket?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-region Write Paths](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To design a high-level solution for rendering remote cursors in a collaborative editor frontend, we need to focus on three main aspects: handling incoming operations efficiently, maintaining local responsiveness, and ensuring smooth updates over WebSocket. The core idea involves using optimistic UI updates, batching operations, and implementing a robust synchronization strategy.

#### Triaging the Question

The interviewer is probing your understanding of real-time collaboration systems, specifically focusing on how to handle remote state changes (cursors) in an efficient and user-friendly manner. They are also interested in your ability to balance performance with complexity, ensuring that the application remains responsive even when dealing with large volumes of data or concurrent users.

#### Assumptions
The editor supports multiple users collaborating simultaneously.
Users can make edits locally, which need to be synchronized back to the server.
Remote cursors are updates from other connected clients.
The system uses WebSocket for real-time communication between the frontend and backend.

#### First Principles Solution

Optimistic UI Updates:

When a remote cursor update is received, we first apply this change locally without waiting for confirmation from the server. This ensures that users see changes immediately, enhancing their experience.
However, if multiple remote updates arrive in quick succession (e.g., due to network latency), we batch these operations and process them together once confirmed by the backend.

Handling Remote Operations:

Each remote update includes an operation object containing details like cursor position and text changes.
Upon receiving a new operation, we apply it to our local document model (e.g., using a diff-based approach) but do not trigger UI updates immediately.
Once all operations are received from the server, we batch them and apply them in one go. This reduces visual flicker and ensures consistency.

Synchronization Strategy:

Implement a conflict resolution mechanism to handle cases where multiple clients make conflicting changes simultaneously (e.g., overlapping cursor positions).
Use timestamps or sequence numbers to determine which operation takes precedence.
Optionally, use optimistic locking or versioning to manage conflicts more gracefully.

WebSocket Communication:

Set up WebSocket connections for real-time communication between the frontend and backend.
The backend aggregates incoming operations and sends periodic batches of changes back to the clients.
Ensure WebSocket handling is optimized for low latency and high throughput.

Performance Optimization:

Use efficient data structures (e.g., immutable objects) to minimize memory overhead during updates.
Implement lazy rendering techniques where only parts of the document that have changed are re-rendered, conserving CPU resources.
Cache frequently accessed data or results of complex operations to avoid redundant computations.

#### Closing Summary
and Common Mistakes:
The key challenge is balancing real-time responsiveness with accurate state synchronization. Optimistic UI updates help in providing an immediate feedback loop, but they must be carefully managed to prevent issues like flicker or inconsistent states.
A common mistake is overcomplicating the synchronization logic, leading to unnecessary complexity and potential performance bottlenecks.
Common Follow-Ups:
How do you handle edge cases where network conditions are poor?
What mechanisms do you have in place for detecting and resolving conflicts between different clients?
Can you discuss how this design scales with increasing numbers of users?

**Comparison:**

```
Operation Type	Description
Cursor Update	Changes the position of a cursor within the document.
Text Change	Edits text at a specific cursor position.
```

**Code:**

```javascript
// Example of applying a remote operation to the local document model (Pseudo-code)
function applyRemoteOperation(operation) {
  const { cursorPosition, text } = operation;
  const cursorIndex = findCursorIndex(cursorPosition);
  if (cursorIndex !== -1) {
    // Apply the text change at the correct position
    documentModel.splice(cursorIndex, 0, ...text.split(''));
  }
}

function handleRemoteUpdates(updates) {
  updates.forEach(applyRemoteOperation); // Apply all operations in one batch

  // Trigger UI update after applying all operations
  renderDocument(documentModel);
}
```

**Example input / output:**

```text
- **Input:** Array of remote updates.
- **Output:** Updated local document model and UI state.

### Example Workflow:
- Client A sends a remote update with cursor position at index 5 and text `"world"`.
- Client B sends an update at index 3 with text `"hello "`.
- Server batches these operations and sends them back to both clients.
- Clients apply the batch of updates in one go, ensuring consistency.
```

**Data:**

```json
{
  "cursorPosition": 10,
  "text": "hello"
}
```

**Flow:**

1. Yes
2. No
3. Receive Remote Updates
4. Is Batch Ready?
5. Apply Operations in Batch
6. Render Document
7. Queue Operation for Later

