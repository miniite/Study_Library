---
type: "practice"
difficulty: "Deep"
---

# Would you push updates to a live dashboard or let it poll, and what are the trade-offs?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Org-wide Backpressure](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between pushing updates to a live dashboard or letting it poll, we need to consider several factors such as latency, network bandwidth, server load, and overall system resilience. Pushing updates typically offers lower latency but may require more complex infrastructure and could introduce additional overhead on the server side. Polling, while generally simpler to implement, can lead to higher latency and increased network traffic.

To understand the trade-offs better, let's break down each approach from first principles:

#### Triage
What Are We Probing?

The question is essentially asking whether we should use a push-based or pull-based architecture for updating a live dashboard. This involves evaluating system performance, resource utilization, and reliability under different usage scenarios.

#### Assumptions
The dashboard is used by multiple clients.
Updates are frequent but not extremely real-time critical.
The client-side (dashboard) has some degree of control over network conditions.
Server resources are a concern.

#### First-Principles Solution
Push-based Update

Advantages:

Lower Latency: Clients receive updates immediately when the server pushes them, reducing user wait times.
Reduced Network Traffic: Only necessary data is sent, potentially saving bandwidth.

Disadvantages:

Increased Server Load: The server needs to maintain more complex communication channels and manage push notifications.
Complexity in Implementation: Requires setting up mechanisms like WebSockets or Server-Sent Events (SSE).
Potential for Downtime: If the network between clients and servers is unstable, push-based updates might fail.
Polling

Advantages:

Simplicity: Easy to implement and manage.
Flexibility in Network Conditions: Clients can handle intermittent connectivity better as they can retry polls.

Disadvantages:

Higher Latency: Users may experience delays between when data changes and when it is displayed on the dashboard.
Increased Bandwidth Usage: Polling often results in more frequent requests, which can consume more bandwidth.
Resource Intensive for Frequent Updates: High-frequency polling can significantly increase server load.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In most scenarios, a hybrid approach might be the best solution. For instance, using WebSockets to push critical updates while falling back to polling in cases of poor network conditions. This balance ensures both efficient use of resources and minimal user wait times.

#### Common mistakes
include underestimating the complexity of implementing a robust push-based system or ignoring the potential for increased server load when relying solely on polling.

#### Follow-ups
might involve discussing specific technologies, such as WebSocket protocols (e.g., JSON-Polling), or strategies to optimize both approaches in real-world applications.

**Comparison:**

```
Approach	Latency	Bandwidth Usage	Complexity
Push	Lower	Moderate	High
Polling	Higher	Higher	Low
```

**Code:**

```python
# Example of setting up a simple WebSocket push mechanism using Flask-SocketIO

from flask import Flask
from flask_socketio import SocketIO, emit

app = Flask(__name__)
socketio = SocketIO(app)

@socketio.on('connect')
def handle_connect():
    print('Client connected')

@app.route('/update')
def update_data():
    # Simulate data update
    socketio.emit('data_update', {'new_value': 42})
    return "Data updated"

if __name__ == '__main__':
    socketio.run(app, debug=True)
```

**Example input / output:**

```

Input:

HTTP
Copy
GET /data HTTP/1.1
Host: example.com

Output (Polling):

JSON
Copy
HTTP/1.1 200 OK
Content-Type: application/json

{
  "timestamp": "2023-10-04T15:00:07Z",
  "value": 42,
  "source": "server_poll"
}

Input (WebSocket Push):

JSON
Copy
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "connected"
}

Output (WebSocket Push - Server Update):

JSON
Copy
{"event": "data_update", "data": {"new_value": 42}}
```

**Data:**

```json
{
  "client": {
    "id": "123",
    "type": "dashboard"
  },
  "update": {
    "timestamp": "2023-10-04T14:56:07Z",
    "value": 42,
    "source": "server_push"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client
4. Poll Dashboard
5. Server Response
6. Client Display Update
7. Network Stable?
8. Re-Poll or Server Push
9. Server Push to Client

