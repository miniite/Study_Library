---
id: "07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/API_Gateway"
tools: ["Prometheus"]
---

# Connection Reuse and Client Errors

**L4 Advanced** · [↑ API Gateway](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Gateway Scaling for Traffic Spikes](<Gateway_Scaling_for_Traffic_Spikes.md>)

## Implementations (tools)

- [Prometheus](<../../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Intermediate] [Your gateway's health check to Kubernetes passes but requests fail because the readiness probe checks the wrong port; how do you design gateway liveness vs readiness probes so a bad probe never blackholes traffic silently?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02808_Your_gateway_s_health_check_to_Kubernetes_passes_but_request.md>)
- [Intermediate] [How would you set up a dashboard that lets you attribute a customer-reported slowness to a specific route, upstream, region, and time window within minutes, without exploding metric cardinality?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02860_How_would_you_set_up_a_dashboard_that_lets_you_attribute_a_c.md>)
- [Deep] [You're on-call and the gateway is returning 503 upstream_reset_before_response_started for 2% of traffic; walk through what this Envoy flag means and how you distinguish a slow upstream from a broken keep-alive.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02791_You_re_on-call_and_the_gateway_is_returning_503_upstream_res.md>)
- [Deep] [How would you set up alerting on the gateway so a single upstream going from 0.1% to 5% error rate pages on-call fast, but a slow drift from 0.1% to 0.3% doesn't; what multi-window burn-rate setup do you use?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02825_How_would_you_set_up_alerting_on_the_gateway_so_a_single_ups.md>)
- [Deep] [How would you implement graceful gateway shutdown so SIGTERM stops new connections, drains in-flight up to a deadline, then force-closes, coordinated with the load balancer's deregistration delay so no request is dropped?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02838_How_would_you_implement_graceful_gateway_shutdown_so_SIGTERM.md>)
- [Deep] [Your gateway routes to a legacy service that only speaks HTTP/1.1 without keep-alive, and the gateway opened a new TCP+TLS connection per request exhausting ephemeral ports; how do you detect and fix connection reuse?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02868_Your_gateway_routes_to_a_legacy_service_that_only_speaks_HTT.md>)
- [Deep] [How would you support long-running async jobs through a gateway using a 202-Accepted + status-URL pattern, and how does the gateway handle routing the polling requests to whichever node holds job state?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02492_How_would_you_support_long-running_async_jobs_through_a_gate.md>)
- [Deep] [You expose WebSocket and SSE endpoints through the gateway alongside REST; how does long-lived connection handling differ, and what timeout and load-balancing pitfalls do you watch for?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02633_You_expose_WebSocket_and_SSE_endpoints_through_the_gateway_a.md>)

## Related concepts

- [TCP and Connection Tracking](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/TCP_and_Connection_Tracking.md>) *(Networking)*
- [Serverless Limits and Throttling](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>) *(Cloud Fundamentals)*
- [Long-running Operations](<../../REST_APIs/Long-running_Operations.md>) *(REST APIs)*

