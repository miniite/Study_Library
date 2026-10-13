---
type: "practice"
difficulty: "Intermediate"
---

# Design a typing indicator and online-status feature for a messaging app.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Notification System](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Notification_System.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a robust typing indicator and online-status feature for a messaging app, we need to focus on efficient user status tracking, real-time communication between clients and servers, and handling various edge cases such as network latency and server load. The core idea is to use push notifications and event-driven architectures to ensure minimal latency.

#### Triaging the Question

The question asks us to design features for a messaging app that include both typing indicators and online statuses. These features are critical for providing real-time user interaction cues, enhancing user experience, and maintaining system reliability. The interviewer is probing your understanding of how to implement these features effectively while considering factors like scalability, performance, and robustness.

#### First-Principles Solution
System Architecture Overview
Client-Side: Each client (e.g., mobile app or web application) will send events to the server when a user starts typing or goes online/offline.
Server-Side: The server will aggregate these events and broadcast them to relevant clients in real-time.
Push Notifications: For long-term offline users, push notifications can be used to update their status.
Detailed Steps

User Authentication and Status Tracking

When a user logs into the app, we store their online/offline status in a database or cache.
Use web sockets or HTTP long polling for real-time communication between clients and servers.

Typing Indicator Implementation

Client detects when a user starts typing by tracking key presses or cursor movement.
When a user types, the client sends a typing event to the server via web socket.

Online Status Update

When a user logs in, their status is set to "online" and sent to the server.
Similarly, when they log out, their status is updated to "offline".

Real-Time Event Handling on Server

The server aggregates typing events and online/offline statuses from all clients.
It broadcasts these updates to relevant clients in real-time using web sockets.

Handling Edge Cases

Network Latency: Use exponential backoff for retransmitting messages if the initial transmission fails.
Server Load: Implement rate limiting on client requests and use caching where possible.
User Disconnection: Maintain a heartbeat mechanism to detect disconnections early.

Push Notifications

For long-term offline users, push notifications can be used to update their status when they come back online.
Use Firebase Cloud Messaging (FCM) for cross-platform support and reliable delivery.

#### Closing Summary

Designing the typing indicator and online-status feature involves a combination of real-time communication protocols, efficient data handling, and robust error management. By focusing on scalable architectures and minimizing latency, we can provide seamless user experiences even under high loads.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring network latency and not implementing proper retransmission strategies.

#### Follow-Up
Discuss how to handle retries using exponential backoff and circuit breakers.
Mistake: Failing to consider server load and not implementing rate limiting or caching.

#### Follow-Up
Explain the importance of load balancing and distributed systems in handling high traffic.
Mistake: Not considering push notifications for long-term offline users.

#### Follow-Up
Discuss alternatives like server-sent events (SSE) or other push notification services.

**Comparison:**

```
Feature	Implementation
Typing Indicator	Client sends typing event to server via web socket; server broadcasts to relevant clients.
Online Status	User login updates status in server cache; broadcast to other clients on status change.
Push Notifications	Use FCM for long-term offline users to update their status when they reconnect.
```

**Code:**

```javascript
// Example Client-Side Code Using Web Sockets
const socket = new WebSocket('ws://example.com/socket');

socket.onopen = () => {
  console.log('Connected to the server');
};

socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  if ('typing' in message) {
    handleTypingIndicator(message.user, message.typing);
  } else if ('status' in message) {
    updateOnlineStatus(message.user, message.status);
  }
};

function startTyping() {
  socket.send(JSON.stringify({ typing: true }));
}

function stopTyping() {
  socket.send(JSON.stringify({ typing: false }));
}
```

**Example input / output:**

```text
Input: User 'Alice' starts typing.
Output: Client broadcasts {user: Alice, typing: true} to the server.
Server aggregates and sends {user: Alice, typing: true} to all clients.

Input: Server receives a heartbeat from user 'Bob'.
Output: Server updates Bob's status to "online" and broadcasts {user: Bob, status: "online"} to all clients.
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "status": "online"},
    {"id": 2, "name": "Bob", "status": "typing"}
  ]
}
```

**Flow:**

1. Online
2. Offline
3. Yes
4. No
5. Yes
6. No
7. User Logs In
8. Status Update?
9. Update Server Cache
10. Broadcast Status to Clients
11. No Action
12. User Starts Typing
13. Send Typing Event?
14. Send {user, typing} to Server
15. Broadcast Typing to Clients
16. Do Nothing
17. Server Receives Status/TypingUpdate
18. Aggregate Updates?
19. Broadcast Aggregated Data toRelevant Clients
20. No Action

