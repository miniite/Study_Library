---
type: "practice"
difficulty: "Deep"
---

# Design a chat-room mediator that must now scale to typing indicators, read receipts, and presence; how do you keep the mediator from becoming a god object as message types multiply?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Pub Sub Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Pub_Sub_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a scalable chat-room mediator that supports typing indicators, read receipts, and presence without making it a "god object," we need to focus on modular design principles, clear separation of concerns, and strategic use of patterns like Observer or Publish-Subscribe. The goal is to ensure the system remains maintainable and extensible as message types increase.

First, let's clarify what the question is asking: We're designing a backend component that handles chat messages in real-time. This component needs to manage various message types including text, images, files, and now also include typing indicators, read receipts, and presence updates. The challenge is to do this without making our mediator class too complex or tightly coupled.

Triaging the Problem

We are probing for understanding of:

Modularity: How can we break down a large system into manageable components.
Design Patterns: Knowledge of patterns like Observer or Publish-Subscribe, which allow decoupling of objects that depend on each other.
Scalability: Ensuring the solution handles increased complexity and message types gracefully.

#### Assumptions
The chat room can have multiple users.
Users can send text messages, images, files, typing indicators, read receipts, and presence updates.
Each type of message has its own handling logic.

#### First-Principles Solution
1. Define a Message Interface

First, define an interface for the messages that the mediator will handle:

JAVASCRIPT
interface ChatMessage {
    id: string;
    timestamp: number;
    senderId: string;
    recipientId: string;
    type: MessageType; // Enum: TEXT, IMAGE, FILE, TYPING, READ_RECEIPT, PRESENCE_UPDATE
}
2. Implement the Mediator

The mediator will act as a central hub for message handling:

JAVASCRIPT
class ChatRoomMediator {

#### cons
tructor(users) {
        this.users = users;
        this.subscriptions = new Map();
    }

    subscribe(user, handler) {
        if (!this.subscriptions.has(user)) {
            this.subscriptions.set(user, []);
        }
        this.subscriptions.get(user).push(handler);
    }

    unsubscribe(user, handler) {

#### cons
t handlers = this.subscriptions.get(user);
        if (handlers && handlers.includes(handler)) {
            handlers.splice(handlers.indexOf(handler), 1);
        }
    }

    handleMessage(message) {
        // Broadcast the message to all subscribers of the relevant user

#### cons
t handlers = this.subscriptions.get(message.senderId);
        if (handlers) {
            for (const handler of handlers) {
                handler(message);
            }
        }
    }
}
3. Implement Handlers for Different Message Types

For each type of message, we implement specific handling logic:

JAVASCRIPT
class TypingHandler {
    handleTypingMessage(message) {

#### cons
ole.log(`${message.senderId} is typing...`);
        // Notify other users that the sender is typing
    }
}

class ReadReceiptHandler {
    handleReadReceipt(message) {

#### cons
ole.log(`Received read receipt for ${message.id}`);
        // Update user state to mark message as read
    }
}

class PresenceUpdateHandler {
    handlePresenceUpdate(message) {

#### cons
ole.log(`${message.senderId} has gone offline`);
        // Notify other users of the sender's presence change
    }
}
4. Integrate Handlers with Mediator

Register these handlers with the mediator so they can be invoked when relevant messages are received:

JAVASCRIPT

#### cons
t mediator = new ChatRoomMediator(users);
mediator.subscribe('user1', handler => {
    if (handler.type === 'TYPING') handleTypingMessage(handler);
    else if (handler.type === 'READ_RECEIPT') handleReadReceipt(handler);
    // Add more conditions for other message types
});

#### Closing Summary
and Common Mistakes/Follow-Ups

By using an Observer pattern, we ensure that the mediator remains clean and focused on core functionality. Each handler is responsible for its own logic, making the codebase modular and easy to extend. This approach also helps in testing individual components independently.

#### Common Mistakes

Tight Coupling: Ensuring handlers don't become tightly coupled with the message object.
Complexity: Not keeping the mediator simple and adding too much complexity early on.
Duplicate Logic: Overlapping logic between different handlers which can lead to bugs and maintenance issues.
Additional Considerations
Performance: Handling a large number of users efficiently, possibly using in-memory caching or database optimizations.
Scalability: Designing the system for future expansion, such as adding more message types or real-time notifications like push notifications.

**Comparison:**

| Message Type | Handler           |
|--------------|-------------------|
| TYPING       | TypingHandler     |
| READ_RECEIPT | ReadReceiptHandler|
| PRESENCE_UPDATE | PresenceUpdateHandler |

**Code:**

```javascript
// Code blocks are already included above
```

**Example input / output:**

```text
Input:
- TypingHandler.subscribe('user1');
- mediator.handleMessage({ id: 'msg1', senderId: 'user1', recipientId: 'user2', type: 'TYPING' });

Output:
- `console.log("user1 is typing...")`
```

**Data:**

```json
{
    "users": [
        { "id": "user1", "name": "Alice" },
        { "id": "user2", "name": "Bob" }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Message Received
8. Is message typed?
9. TypingHandler
10. Is message a read receipt?
11. ReadReceiptHandler
12. Is it presence update?
13. PresenceUpdateHandler
14. Mediator handles genericmessage

