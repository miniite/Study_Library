---
type: "practice"
difficulty: "Basic"
---

# Design a system to store user preferences and settings that sync across devices.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Frontend Graceful Degradation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this design, we will create a robust system to manage and synchronize user preferences and settings across multiple devices. The core idea is to use a server-client architecture where the client app stores local state but regularly syncs with a backend database using RESTful API calls. We'll leverage modern technologies like JWT for secure authentication and Redis for session management.

#### Triage
The primary goal of this system design question is to evaluate your understanding of key components such as data storage, synchronization mechanisms, security considerations, and performance optimization. The interviewer will be probing into how you handle state consistency across devices, secure user sessions, and efficient syncing between the client and server.

#### First-Principles Solution

User Authentication & Session Management: Implement a secure login mechanism using JWT (JSON Web Tokens) for session management. This ensures that only authenticated users can access their settings.
Data Storage: Use a relational database like PostgreSQL to store user preferences and settings in a structured manner, with normalization to avoid redundancy.
Client-Side State Management: Each device will have its local copy of the current user settings. Changes made on one device should be pushed to the server for syncing with other devices.
Synchronization Mechanism: Implement real-time or periodic syncs using WebSocket or a background task in the app that checks for updates and pushes them to the backend.
Error Handling & Retry Logic: Handle network errors gracefully, retry failed sync operations after some delay.
Security Considerations: Ensure data is encrypted both at rest and in transit. Use HTTPS for secure communication.

Detailed Solution:

User Authentication & Session Management

Use JWT to manage user sessions securely. When a user logs in, the server generates a token that includes user information and signs it with a secret key.
Store this token on the client-side (e.g., local storage or secure cookies) for subsequent API calls.

Data Storage

Design a schema for storing user preferences and settings in PostgreSQL:
SQL
CREATE TABLE UserPreferences (
    id SERIAL PRIMARY KEY,
    userId UUID NOT NULL REFERENCES Users(id),
    preferenceName VARCHAR(100) NOT NULL,
    value TEXT NOT NULL,
    createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
Ensure normalization to avoid redundancy, e.g., create a separate table for common preferences and reference them by preferenceId.

Client-Side State Management

Store the user's current settings locally in memory or as JSON objects within local storage.
Whenever a setting is changed on one device, immediately send an update to the server using RESTful API calls.

Synchronization Mechanism

Implement a periodic sync mechanism where each device checks for updates from the server at predefined intervals (e.g., every 5 minutes).
Use WebSocket for real-time updates if network conditions allow.
Design a robust retry logic to handle temporary network issues and ensure consistency.

Error Handling & Retry Logic

Implement error handling in your API calls to catch and log any network or data-related errors.
Introduce exponential backoff for retries, e.g., wait 1 second after the first failure, then double the interval (2 seconds, 4 seconds, etc.).

Security Considerations

Encrypt sensitive user data both at rest and in transit using tools like SSL/TLS for HTTPS connections.
Store passwords securely using strong hashing algorithms and never store plain text passwords.

#### Closing Summary
The key to designing this system is understanding the balance between performance, security, and user experience. By leveraging modern technologies such as JWT and WebSocket, you can ensure seamless and secure synchronization of user settings across devices.

#### Common Mistakes/Follow-Ups

Mistake: Over-complicating the design by implementing complex real-time synchronization mechanisms without considering the impact on server load.

#### Follow-Up
Discuss how to optimize for low-bandwidth environments or when WebSocket is not available. Consider alternatives like scheduled background syncs.

**Comparison:**

```
Column Name	Data Type	Description
id	SERIAL	Unique identifier for each record
userId	UUID	Foreign key referencing Users table
preferenceName	VARCHAR(100)	Name of the preference
value	TEXT	Value of the preference
createdAt	TIMESTAMP	Timestamp when preference was added
updatedAt	TIMESTAMP	Timestamp when preference was last updated
```

**Code:**

```javascript
// Example of a basic RESTful API call using fetch in JavaScript
fetch('/api/preferences', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
    },
    body: JSON.stringify({
        userId: 1,
        preferenceName: 'theme',
        value: 'dark'
    })
})
.then(response => response.json())
.catch(error => console.error('Error:', error));
```

**Example input / output:**

```text
// Example input for setting a user preference
{
    "userId": 1234567890,
    "preferenceName": "theme",
    "value": "dark"
}

// Example output after successful update
{
    "id": 1,
    "userId": 1234567890,
    "preferenceName": "theme",
    "value": "dark",
    "createdAt": "2023-10-01T12:00:00Z",
    "updatedAt": "2023-10-01T12:00:00Z"
}
```

**Data:**

```sql
CREATE TABLE Users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    username VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL
);

CREATE TABLE UserPreferences (
    id SERIAL PRIMARY KEY,
    userId UUID NOT NULL REFERENCES Users(id),
    preferenceName VARCHAR(100) NOT NULL,
    value TEXT NOT NULL,
    createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. User Logs In
5. Is Token Valid?
6. Start Session
7. Fetch User Preferences
8. Store Locally
9. Change Preference?
10. Periodic Sync
11. Send Update to Server
12. Server Updates Database
13. Notify Client about Change

