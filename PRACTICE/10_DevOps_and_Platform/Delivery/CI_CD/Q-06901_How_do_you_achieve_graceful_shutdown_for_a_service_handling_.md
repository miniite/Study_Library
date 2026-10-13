---
type: "practice"
difficulty: "Expert"
---

# How do you achieve graceful shutdown for a service handling long-lived connections (websockets/streams) during a rolling deploy without dropping users mid-session?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Zero_Downtime_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure graceful shutdown of a service handling long-lived connections, such as websockets or streams, during a rolling deploy, we need to focus on several key aspects: maintaining user sessions, coordinating deployments across multiple nodes, and ensuring that users are not dropped mid-session. This is crucial for a seamless upgrade experience without impacting ongoing client interactions.

The core idea here is to manage the service in such a way that new instances can take over connections from old ones without disrupting active sessions. We'll use CI/CD practices to facilitate this by setting up a structured deployment strategy, leveraging load balancers, and implementing connection handoff mechanisms.

**Comparison:**

| Deployment Stage | Action |
|------------------|--------|
| Pre-deploy       | Save state, notify clients. |
| Deploy New Service Version | Monitor health check results. |
| Route Traffic to New Instances | Gradually shift traffic after successful health checks. |

**Code:**

```python
# Example of graceful shutdown logic in Python

def handle_graceful_shutdown(signal):
    # Save state and notify connected clients about the shutdown
    save_state()
    for client in active_clients:
        client.send_shutdown_notification()

def main():
    signal.signal(signal.SIGTERM, handle_graceful_shutdown)
    start_service()

# Example of connection handoff logic

class WebSocketHandler:
    def __init__(self, socket):
        self.socket = socket
        self.is_handover_pending = False

    def on_close(self):
        if not self.is_handover_pending:
            save_connection_state(self.socket)

    def send_shutdown_notification(self):
        # Notify connected client about the impending shutdown
        self.socket.send(b"SHUTDOWN")

# Example of load balancer configuration

load_balancer.configure_health_check(endpoint='/healthcheck')
load_balancer.set_session_persistence('stickiness_cookie_name', 'sessionId')
```

**Example input / output:**

```text
graph TD;
    A[Pre-deploy] --> B[Deploy New Service Version];
    B --> C{Is Health Check Passed?};
    C -- Yes --> D[Mark as Healthy];
    C -- No --> E[Retry Health Check];
    D --> F[Route Traffic to New Instances];
    F --> G[Handle Graceful Shutdown];
    G --> H[Save State and Notify Clients];
    H --> I[New Service Instance Up and Running];
    A --> J[Old Service Instance Still Handling Connections];
```

**Data:**

```json
{
  "service": {
    "name": "WebSocketService",
    "version": "1.2.3"
  },
  "deployment": {
    "strategy": "rolling",
    "max_unavailable": 0.1,
    "min_healthier_replicas": 1
  }
}
```

**Flow:**

1. Yes
2. No
3. Pre-deploy
4. Deploy New Service Version
5. Is Health Check Passed?
6. Mark as Healthy
7. Retry Health Check
8. Route Traffic to New Instances
9. Handle Graceful Shutdown
10. Save State and Notify Clients
11. New Service Instance Up andRunning
12. Old Service Instance StillHandling Connections

