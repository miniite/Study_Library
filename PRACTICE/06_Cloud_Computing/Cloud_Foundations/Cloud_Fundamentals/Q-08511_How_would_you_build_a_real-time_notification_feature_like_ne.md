---
type: "practice"
difficulty: "Intermediate"
---

# How would you build a real-time notification feature (like "new message") in a cloud web app?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Push Communication Patterns](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Push_Communication_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a real-time notification feature like "new message" in a cloud web app, we need to focus on ensuring that users are promptly informed about new updates or events without refreshing the page. The key components involve setting up a server to handle notifications and implementing client-side technologies that can communicate with this server efficiently.

The core idea is to use WebSocket technology for real-time communication between the client and the server. WebSockets provide a full-duplex communication channel over a single TCP connection, allowing both the client and the server to send data at any time. Additionally, we'll need to handle database changes via triggers or polling mechanisms.

The question is probing our understanding of:

Real-time web technologies: WebSocket, Server-Sent Events (SSE), etc.
How to push notifications from a cloud service to multiple clients.
Database change detection and how to integrate it with the real-time mechanism.

#### Assumptions
include:

We're using a cloud provider like AWS or Azure for hosting our application.
Notifications are sent based on specific events, such as new messages in a chat app.

Let's break down the solution step by step:

Database Design: Ensure your database schema supports tracking changes efficiently. For instance, if you have a chat application, you might want to log all messages and track which users have seen them.

#### Example
Schema:
SQL
CREATE TABLE Messages (
    id INT PRIMARY KEY,
    user_id INT,
    content TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE UserNotifications (
    id INT PRIMARY KEY,
    user_id INT,
    message_id INT,
    seen BOOLEAN DEFAULT FALSE
);

WebSocket Implementation: Set up a WebSocket server that can broadcast messages to connected clients.

Code Example:
JAVASCRIPT

#### cons
t io = require('socket.io')(server);

io.on('connection', (socket) => {
    socket.emit('newMessage', { content: 'Connected!' });

    // Handle new message events and push notifications via WebSocket
    app.post('/api/messages', (req, res) => {

#### cons
t message = req.body;
        db.query(`INSERT INTO Messages ...`, (err, result) => {
            if (err) throw err;
            io.emit('newMessage', { content: message.content });
            // Also update UserNotifications for relevant users
        });
    });
});

server.listen(3001, () => console.log('WebSocket Server is running'));

Client-Side Implementation: Use JavaScript to establish a WebSocket connection and listen for new messages.

Code Example:
JAVASCRIPT

#### cons
t socket = io.connect('http://localhost:3001');

socket.on('newMessage', (message) => {

#### cons
ole.log('Received new message:', message);
    // Update UI with the new message
});

Database Triggers or Polling: Implement a mechanism to detect changes in the database and push notifications to WebSocket clients.

#### Example
Using Database Triggers:
SQL
CREATE TRIGGER update_user_notifications AFTER INSERT ON Messages
FOR EACH ROW
BEGIN
    UPDATE UserNotifications SET seen = FALSE WHERE message_id = NEW.id AND user_id IN (SELECT user_id FROM Users);
END;

#### Closing Summary
By integrating WebSocket for real-time communication, tracking database changes effectively, and updating the UI in response to new messages, we can create a seamless notification feature. Common mistakes include not handling disconnections gracefully or failing to update the client-side state correctly.

**Code:**

```

// Example WebSocket Server setup const io = require('socket.io')(server);

io.on('connection', (socket) => { socket.emit('newMessage', { content: 'Connected!' });

CODE
Copy
app.post('/api/messages', (req, res) => {
    const message = req.body;
    db.query(`INSERT INTO Messages ...`, (err, result) => {
        if (err) throw err;
        io.emit('newMessage', { content: message.content });
        // Also update UserNotifications for relevant users
    });
});

});

server.listen(3001, () => console.log('WebSocket Server is running'));

CODE
Copy

```javascript
// Example WebSocket Client setup
const socket = io.connect('http://localhost:3001');

socket.on('newMessage', (message) => {
    console.log('Received new message:', message);
    // Update UI with the new message
});
```

**Example input / output:**

```text
Input: POST /api/messages with JSON { "content": "New Message" }
Output:
1. WebSocket server emits 'newMessage' event.
2. Client receives and updates UI with the new message.
3. Database logs a new message and triggers an update to UserNotifications for relevant users.
```

**Data:**

```sql
CREATE TABLE Messages (
    id INT PRIMARY KEY,
    user_id INT,
    content TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE UserNotifications (
    id INT PRIMARY KEY,
    user_id INT,
    message_id INT,
    seen BOOLEAN DEFAULT FALSE
);
```

