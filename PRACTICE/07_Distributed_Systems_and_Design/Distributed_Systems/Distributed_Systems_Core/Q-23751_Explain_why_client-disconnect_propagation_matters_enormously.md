---
type: "practice"
difficulty: "Deep"
---

# Explain why client-disconnect propagation matters enormously for LLM serving cost: if a client abandons a request, the GPU may keep generating tokens no one will read. How would you design cancellation to propagate through the router and free the KV-cache slot promptly across the fleet?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Load Balancing Stateful Inference](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Load_Balancing_Stateful_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, especially when serving large language models (LLMs), client-disconnect propagation is a critical consideration that impacts cost efficiency significantly. When a client abandons a request mid-stream, it can lead to an uncontrolled generation of tokens on the server side, depleting resources without any benefit. This issue becomes particularly pronounced in GPU-based systems where token generation is resource-intensive.

#### Triaging the Question

The question probes your understanding of how distributed systems manage requests and handle cancellations efficiently. Specifically, it focuses on the mechanics of propagating cancellation signals across a fleet of nodes to ensure that resources are freed promptly when a client disconnects or abandons a request. This involves understanding the architecture, identifying bottlenecks, and proposing solutions.

Key Assumptions
Distributed System: Multiple server nodes are involved in serving LLM requests.
Client-Server Interaction: The system must handle both synchronous and asynchronous client-server interactions.
Resource Management: Efficient management of GPU resources (tokens) is crucial to minimize costs.
Cache Utilization: A key concern is the efficient use of Key-Value (KV) cache slots.

#### First Principles Solution
Step 1: Understanding the Problem

When a client disconnects, the server-side process should be interrupted as quickly as possible. This requires a robust cancellation mechanism that can propagate from the initial entry point (router) to all downstream nodes handling the request.

Step 2: Propagation Mechanism

The solution involves creating a hierarchical or distributed cancellation model where:

Router Layer: The router acts as the first point of contact for client requests and maintains state about ongoing requests.
Worker Nodes: Each node in the worker pool handles parts of the request, such as token generation.
Step 3: Designing Cancellation

To ensure that resources are freed promptly, we need to design a cancellation mechanism:

Client-Side Cancellation Token: The client sends a unique cancellation token with each request.
Router Handling: The router checks for this token upon receiving the initial request and stores it in its state.
Cancellation Signal Propagation:
When the client disconnects, it signals the router to propagate the cancellation signal.
The router then sends a cancellation message to all worker nodes involved in processing the request.
Step 4: Worker Node Implementation

Worker nodes must be designed to listen for and act on cancellation signals. Key steps include:

Polling Mechanism: Nodes periodically check if they have received a cancellation signal.
Graceful Termination: When a node detects a cancellation, it stops generating new tokens and waits for the current token sequence to complete before freeing resources.
Step 5: Cache Slot Management
Cache Tracking: Each worker node tracks which cache slots are in use by associating them with ongoing requests.
Slot Release: Upon receiving a cancellation signal, nodes release their cache slots promptly, allowing other requests to utilize these slots efficiently.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the design of client-disconnect propagation involves creating a robust hierarchical system where the router acts as the central coordinator for request management. Properly designed worker nodes ensure that resources are freed promptly when a client disconnects or abandons a request. Key challenges include ensuring minimal latency in signal propagation and maintaining state consistency across distributed nodes.

#### Common Mistakes

Failure to Propagate Cancellation Signals: Not having a mechanism to propagate cancellation signals from the router down to worker nodes.
Inefficient Cache Management: Failing to track and release cache slots promptly, leading to resource wastage.
Resource Locking: Holding onto resources longer than necessary due to uncontrolled token generation.

#### Follow-Ups

How would you implement a timeout mechanism for client requests?
What measures can be taken to reduce latency in cancellation signal propagation?

**Comparison:**

```
Role	Responsibilities
Router	Manages request state and propagates cancel signals.
Worker Node	Processes requests and handles cancellation signals.
```

**Code:**

```python
# Example of a simple router implementation with cancellation support

class RequestRouter:
    def __init__(self):
        self.requests = {}

    def handle_request(self, request_id, token):
        self.requests[request_id] = {'token': token}

    def propagate_cancellation(self, request_id):
        if request_id in self.requests:
            for worker_node in self.worker_nodes:
                worker_node.cancel(request_id)
```

**Example input / output:**

```text
Input:
- Client disconnects during a request with ID 123

Router Actions:
1. Detect client disconnection and signal cancellation.
2. Propagate cancellation to Worker Nodes.

Worker Node Actions (Node 1):
1. Check for token "abc".
2. Stop generating tokens and free cache slot.

Worker Node Actions (Node 2):
1. Check for token "def" or "abc".
2. Continue processing if token not found, otherwise stop.
```

**Data:**

```json
{
  "requests": {
    "123": {"status": "active", "token": "abc"},
    "456": {"status": "cancelled"}
  },
  "workers": [
    {"id": 1, "tokens": ["abc"], "status": "ready"},
    {"id": 2, "tokens": ["def"], "status": "processing"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Client Disconnect
6. Router Detects Disconnection
7. Propagate Cancellation Signalto Worker Nodes
8. Worker Node 1 Checks Token
9. Token Found?
10. Stop Generating Tokens
11. Continue Processing
12. Worker Node 2 Checks Token
13. Token Found?
14. Stop Generating Tokens
15. Continue Processing

