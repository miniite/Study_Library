---
type: "practice"
difficulty: "Deep"
---

# You're on-call and the gateway is returning 503 upstream_reset_before_response_started for 2% of traffic; walk through what this Envoy flag means and how you distinguish a slow upstream from a broken keep-alive.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Connection Reuse and Client Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The upstream_reset_before_response_started error in an API Gateway like Envoy indicates that the upstream server terminated the connection before it was able to send the first byte of the response. This can occur due to a slow or unresponsive downstream service, or because the keep-alive mechanism is misconfigured. To diagnose and resolve this issue, we need to differentiate between a slow upstream and a broken keep-alive scenario.

First, let's understand the context: Envoy uses HTTP/2 by default with keep-alive connections to reuse TCP connections for multiple requests. The upstream_reset_before_response_started error specifically highlights that the connection was reset (likely due to a timeout or abrupt termination) before the response started being sent from the upstream server.

To distinguish between slow upstream and broken keep-alive, we can follow these steps:

Check Keep-Alive Settings: Verify the keep-alive settings in your Envoy configuration. If the keep-alive idle time is set too low, it might lead to connections being prematurely closed due to inactivity.
Analyze Upstream Service Performance: Monitor the performance of the upstream service for slow responses or crashes that could cause premature connection resets.
Envoy Logs and Metrics: Examine Envoy logs to identify patterns related to the 503 errors. Check if there are any common factors among the requests leading to these errors, such as specific path patterns or time of day.
Upstream Response Time Distribution: Use tools like Prometheus or Grafana to analyze the response times from your upstream service. This can help identify if some requests are indeed taking too long and causing connection resets.

Let's go through a step-by-step approach:

Review Envoy Configuration:

YAML
# Example configuration snippet for an HTTP connection manager
http_connection_manager:
  stat_prefix: "http"
  route_config:
    name: "service_route"
    virtual_hosts:
      - name: "upstream_service"
        domains: ["*"]
        routes:
          - match: { prefix: "/" }
            route:
              cluster: "upstream_cluster"

Check Keep-Alive Settings: Ensure that the keep-alive settings are appropriate for your use case. For example, if you have a resource-intensive upstream service, you might need to increase the keepalive_timeout in Envoy.

YAML
http_connection_manager:
  stat_prefix: "http"
  route_config:
    name: "service_route"
    virtual_hosts:
      - name: "upstream_service"
        domains: ["*"]
        routes:
          - match: { prefix: "/" }
            route:
              cluster: "upstream_cluster"
connection pool config:
  http2_protocol_options:
    keepalive_time_ms: 30000

Monitor and Analyze Upstream Service: Use tools like Prometheus to monitor the response time of your upstream service.

BASH
# Example Prometheus query to get HTTP request duration
sum(rate(http_request_duration_seconds_sum[1m])) by (instance)

Analyze Envoy Logs: Check for patterns in the logs that might indicate slow or unresponsive upstreams.

PLAINTEXT
2023-10-05T14:39:27.018Z error "upstream reset before response started" method="HTTP/1.1 POST /api/v1/slow-service" route="[::1]:10000 -> [::1]:8080" cluster=upstream_cluster

#### Triage
and Testing:

Slow Upstream Scenario: If the upstream service is consistently slow, consider optimizing its performance or adjusting the keep-alive timeout.
Broken Keep-Alive Scenario: If there are no patterns of slow response times but consistent connection resets, it might be due to a misconfiguration in Envoy or an issue with the keep-alive mechanism.

By following these steps and continuously monitoring both the upstream service and Envoy configuration, you can effectively distinguish between slow upstreams and broken keep-alive issues and take appropriate corrective actions.

**Comparison:**

```
Envoy Configuration	Value
Route Config Name	service_route
Virtual Hosts Name	upstream_service
Domains	*
Routes Prefix	/
Cluster	upstream_cluster
Keep-Alive Time	30000 ms
```

**Code:**

```yaml
# Example Envoy config snippet for HTTP connection manager
http_connection_manager:
  stat_prefix: "http"
  route_config:
    name: "service_route"
    virtual_hosts:
      - name: "upstream_service"
        domains: ["*"]
        routes:
          - match: { prefix: "/" }
            route:
              cluster: "upstream_cluster"
  connection pool config:
    http2_protocol_options:
      keepalive_time_ms: 30000
```

**Example input / output:**

```text
# Example Envoy logs
2023-10-05T14:39:27.018Z error "upstream reset before response started" method="HTTP/1.1 POST /api/v1/slow-service" route="[::1]:10000 -> [::1]:8080" cluster=upstream_cluster
```

**Data:**

```json
{
  "route_config": {
    "name": "service_route",
    "virtual_hosts": [
      {
        "name": "upstream_service",
        "domains": ["*"],
        "routes": [
          {
            "match": { "prefix": "/" },
            "route": { "cluster": "upstream_cluster" }
          }
        ]
      }
    ]
  },
  "connection_pool_config": {
    "http2_protocol_options": {
      "keepalive_time_ms": 30000
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Upstream Slow?
5. Optimize Upstream Service
6. Trial with Increased Keep-AliveTimeout
7. Monitor Performance and Logs
8. Continue Monitoring andAdjustments as Needed

