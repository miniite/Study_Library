---
type: "practice"
difficulty: "Deep"
---

# How would you implement graceful gateway shutdown so SIGTERM stops new connections, drains in-flight up to a deadline, then force-closes, coordinated with the load balancer's deregistration delay so no request is dropped?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Connection Reuse and Client Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a graceful shutdown of an API Gateway when receiving a SIGTERM signal, we need to implement a mechanism that stops accepting new connections while draining any in-flight requests up to a specified deadline, and then force-closes them after the deregistration delay with the load balancer. This involves coordinated actions between the gateway, application, and load balancer.

#### Triage
What's Being Probed

The question is probing our understanding of how to handle shutdown signals in a complex system where multiple components need to coordinate their operations. The interviewer is likely interested in:

How to manage the lifecycle of connections gracefully.
The implementation details of handling SIGTERM and other relevant signals.
Coordination mechanisms with external systems like load balancers.

#### Assumptions
We are using a typical HTTP/2 API Gateway running on a multi-process or multi-threaded application.
The load balancer supports deregistration delays, allowing for gradual removal of instances without immediate interruption of requests.
We have access to logging and monitoring tools for debugging and ensuring the process runs as expected.

#### First-Principles Solution
Signal Handling: Implement a signal handler in your application to catch SIGTERM. This handler will initiate the shutdown sequence.
Connection Drain: Gracefully stop accepting new incoming connections, but allow existing in-flight requests to complete.
Drain Timeout: Set a timeout for draining existing requests and notify the load balancer about the deregistration delay.
Force-Close: After the drain period, forcefully close all remaining connections.
Step-by-Step Implementation

Signal Handler

PYTHON
import signal
import time
from http.server import HTTPServer, BaseHTTPRequestHandler

class GracefulShutdownHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'Hello, world!')

    def handle_timeout(self):
        # Implement logic to notify the load balancer
        print("Notifying load balancer about deregistration delay")
        time.sleep(3)  # Simulate delay

def shutdown_gracefully():
    server.socket.shutdown(socket.SHUT_RDWR)
    server.socket.close()
    print("Graceful shutdown complete")

signal.signal(signal.SIGTERM, lambda signum, stack_frame: shutdown_gracefully())

if __name__ == "__main__":
    httpd = HTTPServer(('localhost', 8000), GracefulShutdownHandler)
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("Shutting down...")

Connection Drain

Use a custom connection pool that respects the drain period.
Track in-flight requests and notify the load balancer when they complete.

Drain Timeout and Load Balancer Coordination

PYTHON
import threading

class GracefulShutdownHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'Hello, world!')

    def handle_timeout(self):
        # Notify load balancer about deregistration delay
        print("Notifying load balancer about deregistration delay")
        time.sleep(3)  # Simulate delay

def shutdown_gracefully():
    server.socket.shutdown(socket.SHUT_RDWR)
    server.socket.close()
    print("Graceful shutdown complete")

signal.signal(signal.SIGTERM, lambda signum, stack_frame: shutdown_gracefully())

if __name__ == "__main__":
    httpd = HTTPServer(('localhost', 8000), GracefulShutdownHandler)

    # Simulate graceful shutdown
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("Shutting down...")

        # Drain existing connections
        for i in range(10):  # Simulating 10 in-flight requests
            threading.Thread(target=httpd.handle_timeout).start()

        time.sleep(3)  # Allow drain period

        # Force-close remaining connections after drain timeout
        shutdown_gracefully()

Force-Close

Ensure all remaining connections are forcefully closed to prevent any data loss or corruption.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key is to implement a signal handler that gracefully stops accepting new requests while draining existing in-flight requests within a specified timeout period. The shutdown process should also coordinate with the load balancer's deregistration delay to ensure no request is dropped. Common mistakes include:

Ignoring signal handling mechanisms.
Not respecting the drain timeout, which can result in abrupt termination of connections.

#### Follow-up
questions might involve discussing specific implementations for different cloud providers or load balancers, or strategies for handling different types of requests (e.g., long-running tasks).

**Code:**

```python
import signal
import time
from http.server import HTTPServer, BaseHTTPRequestHandler

class GracefulShutdownHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'Hello, world!')

    def handle_timeout(self):
        print("Notifying load balancer about deregistration delay")
        time.sleep(3)  # Simulate delay

def shutdown_gracefully():
    server.socket.shutdown(socket.SHUT_RDWR)
    server.socket.close()
    print("Graceful shutdown complete")

signal.signal(signal.SIGTERM, lambda signum, stack_frame: shutdown_gracefully())

if __name__ == "__main__":
    httpd = HTTPServer(('localhost', 8000), GracefulShutdownHandler)

    # Simulate graceful shutdown
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("Shutting down...")

        for i in range(10):  # Simulating 10 in-flight requests
            threading.Thread(target=httpd.handle_timeout).start()

        time.sleep(3)  # Allow drain period

        shutdown_gracefully()
```

