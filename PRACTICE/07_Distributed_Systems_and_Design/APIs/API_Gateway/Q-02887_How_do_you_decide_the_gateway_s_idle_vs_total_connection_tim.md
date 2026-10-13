---
type: "practice"
difficulty: "Deep"
---

# How do you decide the gateway's idle vs total connection timeouts to defend against slowloris-style slow clients without cutting off a legitimately slow report download; how do these timeouts interact?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Timeouts Retries and Backoff](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Timeouts_Retries_and_Backoff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on the idle and total connection timeouts for an API Gateway, we need to balance defense against slowloris-style attacks (where slow clients flood the server with a large number of connections that never complete their requests) while ensuring legitimate slow report downloads can still proceed. The key is to understand how these timeouts interact and configure them appropriately.

#### Triaging the Question

The question revolves around setting connection timeouts in an API Gateway to prevent DDoS attacks, specifically those using the slowloris technique. We need to ensure that the configuration allows for long-running reports while defending against attackers who might open many connections but never complete their requests.

#### Assumptions
and Context
Gateway: An API Gateway like NGINX or AWS API Gateway.
Slowloris Attack: The attacker opens many connections, sending partial HTTP requests to keep them alive indefinitely.
Report Downloads: Legitimate users who take a long time to download reports because of their size.

#### First-Principles Solution

To address this, we need to set the appropriate timeouts for both idle and total connection durations. Here’s how:

Idle Timeout: This is the period after which a connection is considered inactive and can be closed if there's no activity.
Total Timeout: The maximum duration of time that a single request can take before being terminated.
Idle Timeout
Objective: Prevent slowloris attacks by closing connections that are not active for an extended period.
Value: Set to a value like 5 minutes (300 seconds) or more. This allows legitimate long-running downloads but limits the number of connections that can be held open indefinitely.
Total Timeout
Objective: Ensure requests don't stall indefinitely, protecting against slowloris and other types of attacks.
Value: Set to a value like 15 minutes (900 seconds). This gives users plenty of time for large report downloads while still being secure.
Interactions Between Timeouts

The idle timeout applies after the connection is established. If there's no activity, the gateway will close the connection. The total timeout applies during the request itself and ensures that a single request cannot take too long to complete.

Full Solution

Here’s an example configuration for NGINX:

NGINX
http {
    # Define the limits
    limit_req_zone $binary_remote_addr zone=one:10m rate=5r/m;
    limit_conn_zone $binary_remote_addr zone=two:10m;

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Connection "";

            # Set idle and total timeouts
            send_timeout 300;  # Idle timeout: close inactive connections after 5 minutes
            client_max_body_size 10G;  # Handle large reports

            # Other configuration options...
        }
    }
}

#### Closing Summary
+ Common Mistakes/Follow-Ups
Mistake: Setting both timeouts too low can cut off legitimate users. Ensure there’s a balance.
Mistake: Not setting any idle timeout can allow slowloris attacks to flood your server with many connections, but no activity.

#### Follow-Up
Consider using rate limiting (as shown in the example) to further mitigate slowloris attacks.

By understanding and configuring these timeouts carefully, you can defend against slowloris-style attacks while ensuring that legitimate long-running reports are not interrupted. Always test thoroughly under various conditions to ensure your configuration works as expected.

