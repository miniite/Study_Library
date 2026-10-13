---
type: "practice"
difficulty: "Intermediate"
---

# What is a health check and how does a load balancer use it to route traffic?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Health Checks and Load Balancers](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Health_Checks_and_Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Health checks are crucial for ensuring that your backend services are up and running before directing any traffic to them. In this context, a load balancer uses health checks to monitor the availability of backend servers or instances, dynamically routing traffic based on their state.

To understand how load balancers use health checks, let's first define what we mean by "health check." At its core, a health check is a mechanism that periodically tests if an application or service can respond correctly. Load balancers employ these checks to determine whether a server should receive new requests.

#### Triage
What the Question Is Really Asking and Assumptions

The question is asking us to explain how load balancers use health checks in their operations. The main assumptions here are:

You need to have a basic understanding of load balancing.
Health checks are a mechanism used by load balancers to ensure backend services are operational.
Step-by-Step Solution: From First Principles
1. Understanding Load Balancing

Load balancing is the process of distributing incoming network traffic across multiple servers to optimize resource utilization, maximize throughput, and minimize response time. It helps in achieving high availability and scalability by ensuring that no single server bears the entire load.

2. The Role of Health Checks

Health checks are essential for maintaining reliable and efficient service delivery. Here’s how they work:

Initial Health Check: When a new backend instance is added to the load balancer, it performs an initial health check to ensure that the instance is ready to handle traffic.
Periodic Health Checks: Once active, load balancers continuously perform periodic health checks on all instances in their pool to monitor their availability and responsiveness.
3. Health Check Mechanisms

Load balancers can use various mechanisms for performing health checks:

HTTP/HTTPS: Sending HTTP or HTTPS requests to a specific endpoint (e.g., /health).
TCP: Establishing a TCP connection and verifying that the server responds.
ICMP: Using ping commands to test connectivity.
4. Health Check Responses

When a backend instance fails a health check, the load balancer marks it as unhealthy:

The load balancer stops routing traffic to this instance until it passes subsequent checks.
This can help in isolating faulty instances and ensuring that only healthy servers handle live traffic.
5. Advanced Health Checks

For more sophisticated scenarios, load balancers might implement advanced health check strategies such as:

Custom Scripts: Running custom scripts on the backend server to verify its operational state.
Heartbeat Monitoring: Monitoring heartbeat messages between the load balancer and backend instances.
Failure Detection Mechanisms: Configuring thresholds for timeouts or response delays.

#### Example
of a Health Check Mechanism

Let’s consider an example where we use HTTP health checks:

Initial Setup:

PLAINTEXT
Load Balancer -> Backend Server (HTTP GET to /health)

Health Check Success:

Response: 200 OK with expected content.
Action: Mark as healthy and route traffic.

Health Check Failure:

Response: Timeout or invalid status code (e.g., 500 Internal Server Error).
Action: Mark as unhealthy, stop routing traffic.

#### Closing Summary

In summary, health checks are a vital part of load balancing systems. They ensure that only healthy backend instances receive incoming traffic, improving overall system reliability and performance. By continuously monitoring the state of backend services, load balancers can dynamically adjust traffic distribution to maintain optimal service levels.

#### Common Mistakes/Follow-Ups
Ignoring Health Checks: Failing to implement or configure health checks properly can lead to routing traffic to unhealthy instances.
Over-reliance on Simple Checks: Relying too heavily on basic HTTP/HTTPS health checks might not catch all issues, leading to intermittent failures.
Triaging Further Questions:
How do different cloud providers handle health checks in their load balancers?
Are there any trade-offs between the complexity of health check mechanisms and system performance?

**Comparison:**

```
Health Check Mechanism	Description
HTTP/HTTPS	Send a request to a known endpoint and check the response status code.
TCP	Establish a connection and ensure the server responds correctly.
ICMP	Ping commands to test network connectivity.
```

**Code:**

```python
# Example Python code for performing a simple HTTP health check
import requests

def perform_health_check(url):
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return True, response.text
        else:
            return False, f"HTTP Status: {response.status_code}"
    except Exception as e:
        return False, str(e)

# Usage example
is_healthy, health_info = perform_health_check("http://example.com/health")
print(f"Health Check Result: {'Healthy' if is_healthy else 'Unhealthy'} - {health_info}")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `http://example.com/health` |
| **Request** | HTTP GET to `/health` |
| **Response** (Success) | 200 OK, response text = "Service is running" |
| **Response** (Failure) | 500 Internal Server Error |

**Data:**

```json
{
  "endpoint": "/health",
  "status_codes": [200],
  "timeout": 5,
  "retries": 3
}
```

**Flow:**

1. 200 OK
2. Non-200
3. Initial Health Check
4. Perform HTTP GET
5. Response Status Code
6. Mark as Healthy
7. Mark as Unhealthy, StopRouting Traffic

