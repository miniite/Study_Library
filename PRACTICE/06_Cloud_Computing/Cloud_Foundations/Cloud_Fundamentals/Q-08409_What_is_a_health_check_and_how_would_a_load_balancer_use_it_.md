---
type: "practice"
difficulty: "Basic"
---

# What is a health check and how would a load balancer use it to route around a crashed inference server?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Health Checks and Load Balancers](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Health_Checks_and_Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Health checks are essential mechanisms in cloud environments, ensuring that services remain available by periodically assessing their operational status. For an AI engineer at any level, understanding how load balancers use health checks to route traffic around a crashed inference server is crucial. At a basic concept level, we will focus on the fundamental aspects of health checks and how they are applied within a load balancing scenario.

Load balancers are used to distribute network traffic across multiple servers for reliability, performance, and availability. A health check in this context involves sending specific requests to each server to determine if it is functioning correctly. If a server fails the health check, the load balancer routes traffic away from that server until it passes the health checks again.

To break down the question step by step using first principles:

Understanding Health Checks: A health check typically involves sending a predefined request to an endpoint of the server. The response is then analyzed to determine if the service is operational.
Load Balancer Operation: Load balancers receive incoming requests and distribute them across available servers based on specific criteria (e.g., round-robin, least connections).
Health Check Mechanism: When a health check fails for a server, the load balancer marks that server as unhealthy or down and stops routing traffic to it.
Recovery: Once the server passes the health checks again, the load balancer will start sending traffic back to it.

Let’s reason through this with an example:

Suppose we have three inference servers: Server A, B, and C.
Each server has a REST endpoint /health that returns 200 OK when healthy and 503 Service Unavailable when unhealthy.
The load balancer sends periodic requests to each server’s /health endpoint.
If any server returns 503, the load balancer stops sending traffic to it.

Now, let's illustrate this with a more detailed example:

#### Example
A Load Balancer Configuring Health Checks

Server Status:

Server A: Healthy (200 OK)
Server B: Unhealthy (503 Service Unavailable)
Server C: Healthy (200 OK)

Load Balancer Actions:

The load balancer sends requests to each server’s /health endpoint.
If a request fails (e.g., Server B returns 503), the load balancer marks Server B as unhealthy and stops routing traffic to it.

#### Closing Summary

Health checks are vital for maintaining service availability by ensuring that only healthy servers receive incoming requests. In the context of a load balancer, health checks help in dynamically routing around crashed or failing inference servers, thereby enhancing overall system reliability.

#### Common Mistakes/Follow-Ups
Failing to Configure Health Checks: Not setting up proper health check mechanisms can lead to prolonged outages.
Misconfigured Endpoints: Incorrectly configured health endpoints may result in false positives (indicating healthy services as unhealthy).
Over-reliance on Single Health Check Endpoint: Relying solely on a single endpoint can make the system vulnerable if that endpoint itself fails.

**Comparison:**

| IP           | Health Status |
|--------------|---------------|
| 192.168.1.1  | True          |
| 192.168.1.2  | False         |
| 192.168.1.3  | True          |

**Code:**

```python
# Example of a simple health check function in Python
def check_health(server):
    response = requests.get(f"{server}/health")
    return response.status_code == 200

servers = ["http://192.168.1.1", "http://192.168.1.2", "http://192.168.1.3"]
healthy_servers = [server for server in servers if check_health(server)]
```

**Example input / output:**

```text
- **Input**: List of servers with their health statuses.
- **Output**: List of healthy servers.

| Servers         | Health Status |
|-----------------|---------------|
| 192.168.1.1     | True          |
| 192.168.1.2     | False         |
| 192.168.1.3     | True          |
```

**Data:**

```json
{
  "servers": [
    {"ip": "192.168.1.1", "healthStatus": true},
    {"ip": "192.168.1.2", "healthStatus": false},
    {"ip": "192.168.1.3", "healthStatus": true}
  ]
}
```

**Flow:**

1. Healthy
2. Unhealthy
3. Healthy
4. Unhealthy
5. Healthy
6. Unhealthy
7. Request to Load Balancer
8. Server A Health Status?
9. Route Traffic to Server A
10. Do Not Route Traffic to ServerA
11. Server B Health Status?
12. Server B Health Status?
13. Route Traffic to Server B
14. Do Not Route Traffic to ServerB
15. Server C Health Status?
16. Server C Health Status?
17. Route Traffic to Server C
18. Do Not Route Traffic to ServerC
19. Load Balancer Decisions

