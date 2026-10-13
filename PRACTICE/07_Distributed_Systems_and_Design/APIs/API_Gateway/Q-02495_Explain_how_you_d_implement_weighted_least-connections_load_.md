---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd implement weighted least-connections load balancing at the gateway and why it can beat round-robin when upstream instances have heterogeneous capacity or in-flight long requests.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Load Balancing and Zone Aware Routing](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Weighted least-connections (WLC) load balancing is a strategy that assigns connections to backend servers based on their current load, ensuring that less busy servers handle more requests. This approach can outperform round-robin when upstream instances have varying capacities or when long-pending requests tie up server resources. By distributing traffic more intelligently, WLC improves overall system performance and reduces the likelihood of overloading any single instance.

#### Triaging the Question

The question is essentially asking how to implement weighted least-connections load balancing at an API Gateway level, which involves understanding backend server capacities and distributing incoming requests accordingly. The interviewer is probing your knowledge of load balancing techniques, your ability to handle heterogeneous resources efficiently, and your understanding of practical trade-offs in system design.

#### Assumptions
We have multiple upstream servers with varying processing capabilities.
Long-pending requests can tie up a server for extended periods.
The goal is to ensure that no single server becomes a bottleneck.
Each backend server has some degree of health monitoring and availability status.

#### First-Principles Solution
Step 1: Health Monitoring

Before distributing any request, the API Gateway must verify that the upstream servers are healthy and available. This can be done using periodic health checks or by receiving notifications from a health check service. If an instance is down, it should not receive any new requests.

API Gateway
Health Check Service
Healthy Backend Server
Downed Backend Server
Step 2: Load Calculation

For each backend server, calculate the current load by counting active connections. This can be done using a custom metric or leveraged from a health check service that provides this information.

PYTHON
def get_load(server_id):
    # Assume we have access to some health check service that returns connection count
    return health_check_service.get_active_connections(server_id)
Step 3: Weighted Selection

Based on the calculated loads, select the server with the least active connections. The weight here is inversely proportional to the number of active connections; a server with fewer connections has more "weight" and thus higher probability of receiving a request.

PYTHON
def weighted_least_connections(backend_servers):
    min_load = float('inf')
    selected_server = None

    for server in backend_servers:
        load = get_load(server.id)
        if load < min_load:
            min_load = load
            selected_server = server

    return selected_server
Step 4: Request Distribution

Once the server with the least active connections is chosen, route the request to that server. This can be done using a routing table or by directly setting headers in the API Gateway's responses.

PYTHON
def distribute_request(request, backend_servers):
    selected_server = weighted_least_connections(backend_servers)
    # Route the request to the selected server
    return f"Routing request to {selected_server.id}"
Step 5: Handling Long-Pending Requests

To mitigate the impact of long-pending requests on the overall system, you can implement a timeout mechanism. If a backend server is taking too long to respond, the API Gateway should abandon that connection and route the next request to another server.

PYTHON
def handle_request(request):
    start_time = time.time()
    response = send_request_to_backend(request)

    if (time.time() - start_time) > MAX_RESPONSE_TIME:
        return "Abandoning long-pending request"
    else:
        return response

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, implementing weighted least-connections load balancing involves health monitoring, dynamic load calculation, and intelligent request routing. This method ensures that the backend servers with less active connections handle more requests, thus preventing any single server from becoming a bottleneck.

#### Common mistakes
include not properly handling long-pending requests, failing to implement accurate health checks, or not considering edge cases where servers might become unresponsive without immediate detection.

#### Follow-ups
might involve discussing how this approach scales under high traffic conditions, how to integrate with existing monitoring tools, and whether there are any downsides to this method compared to round-robin.

**Comparison:**

```
ID	Active Connections
1	2
2	4
3	1
```

**Code:**

```python
# Example code for weighted least-connections load balancing
from typing import List

class BackendServer:
    def __init__(self, id: int):
        self.id = id
        self.active_connections = 0

def get_load(server_id: int) -> int:
    # Simulate a health check service that returns active connections count
    return 5 - server_id  # For example purposes

def weighted_least_connections(backend_servers: List[BackendServer]) -> BackendServer:
    min_load = float('inf')
    selected_server = None

    for server in backend_servers:
        load = get_load(server.id)
        if load < min_load:
            min_load = load
            selected_server = server

    return selected_server

def distribute_request(request, backend_servers: List[BackendServer]) -> str:
    selected_server = weighted_least_connections(backend_servers)
    # Simulate routing request to the selected server
    return f"Routing request to {selected_server.id}"

def handle_request(request) -> str:
    start_time = time.time()
    response = send_request_to_backend(request)

    if (time.time() - start_time) > MAX_RESPONSE_TIME:
        return "Abandoning long-pending request"
    else:
        return response

MAX_RESPONSE_TIME = 10  # Example timeout
```

**Example input / output:**

|Request| Backend Server ID|
|---|---|
|Sample request| 3|

**Data:**

```

[ {"id": 1, "active_connections": 2}, {"id": 2, "active_connections": 4}, {"id": 3, "active_connections": 1} ]

CODE
Copy

### Input-Output
Example input and output scenarios:

#### Input
```python
backend_servers = [BackendServer(1), BackendServer(2), BackendServer(3)]
request = "Sample request"
Output
PYTHON
Copy
Routing request to 3
```

**Flow:**

1. Server is Healthy
2. Server is Downed
3. API Gateway
4. Health Check
5. Select Least Load
6. Do Not Route
7. Calculate Active Connectionsfor Each Backend Server
8. Find Minimum Load
9. Route Request to SelectedServer

