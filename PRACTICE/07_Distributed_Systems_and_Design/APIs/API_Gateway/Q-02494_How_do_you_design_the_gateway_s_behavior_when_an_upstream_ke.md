---
type: "practice"
difficulty: "Expert"
---

# How do you design the gateway's behavior when an upstream keeps a connection open but sends bytes very slowly (slowloris-style, or a genuinely slow report); how do idle vs total timeouts and read deadlines differ and interact?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Timeouts Retries and Backoff](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Timeouts_Retries_and_Backoff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the scenario where an API Gateway must manage upstream services that keep connections open but send bytes very slowly (slowloris-style attacks or slow reports), we need to carefully design our gateway's behavior around connection timeouts, idle timeouts, and read deadlines. The key is understanding how these settings interact and their implications on performance and security.

#### Triaging the Question

The question revolves around ensuring that your API Gateway can handle slow upstream services without being overwhelmed by long-lived connections or slow data transmission rates. Specifically, it probes your knowledge of connection management, timeout policies, and how to balance resource usage with the need for robust service delivery.

#### First-Principles Solution

Connection Handling:

The gateway should manage the overall duration a client can maintain an open connection by setting total timeouts.
For slowloris-style attacks or genuinely slow reports, we might allow connections to remain open but limit how long the gateway waits for more data.

Idle vs Total Timeouts:

Total Timeout: This is the maximum duration a client can keep an open connection with no activity (read/write). It ensures that even if an upstream sends very slow responses, the connection won't be held indefinitely.
Idle Timeout: This is the period of inactivity before the gateway closes the connection. In this context, it's more relevant for managing resources and preventing connections from being hijacked.

Read Deadline:

A read deadline ensures that if an upstream service takes too long to send data (e.g., due to slowloris), the gateway can close the connection without waiting indefinitely.
Detailed Explanation

Total Timeout:

Set a reasonable total timeout value, which is the maximum duration a client can keep an open connection.
For example, setting it to 30 seconds ensures that connections are closed after 30 seconds of inactivity or slow response from the upstream.

Idle Timeout:

The idle timeout should be set shorter than the total timeout to ensure that idle connections are managed efficiently.
A value like 5-10 seconds can prevent unnecessary resource consumption while still allowing for some variability in slow responses.

Read Deadline:

Implement a read deadline to handle cases where an upstream service sends data very slowly.
For instance, if the gateway expects a response within 2 seconds, setting a read deadline of 1 second ensures that it doesn't wait indefinitely.
Interaction Between Settings

Total Timeout and Idle Timeout:

The total timeout acts as a safety net to ensure connections are closed if they remain idle for too long.
The idle timeout helps manage the number of open connections more aggressively, reducing resource usage in scenarios with many slow requests.

Read Deadline and Total/Idle Timeouts:

Read deadlines provide immediate action when an upstream is unresponsive or sending data very slowly.
They can complement total and idle timeouts by ensuring that even if a connection remains open due to inactivity, the gateway doesn’t wait indefinitely for slow responses.
Practical Implementation

Here’s a simplified configuration example using NGINX as the API Gateway:

NGINX
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Connection "";

            # Total timeout (30 seconds)
            proxy_read_timeout 30s;
            proxy_connect_timeout 5s;
            proxy_send_timeout 10s;

            # Idle timeout (5 seconds)
            keepalive_requests 20; # Allows up to 20 requests per connection
        }
    }
}

#### Closing Summary
and Common Mistakes
Mistake 1: Not setting any timeouts can lead to resource exhaustion from long-lived connections.
Mistake 2: Setting overly aggressive timeouts can result in prematurely closing legitimate slow responses, impacting user experience.
Mistake 3: Ignoring read deadlines can leave the gateway waiting indefinitely for unresponsive upstream services.

In conclusion, a well-designed API Gateway should balance security and performance by setting appropriate total, idle, and read deadlines. This ensures that slowloris-style attacks or genuinely slow reports are handled gracefully without overwhelming system resources.

**Code:**

```
NGINX
Copy
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Connection "";

            # Total timeout (30 seconds)
            proxy_read_timeout 30s;
            proxy_connect_timeout 5s;
            proxy_send_timeout 10s;

            # Idle timeout (5 seconds)
            keepalive_requests 20; # Allows up to 20 requests per connection
        }
    }
}
```

