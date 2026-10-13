---
id: "07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Resilience_to_Dependencies"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/APIs/API_Gateway"
tools: ["Redis"]
---

# Gateway Resilience to Dependencies

**L5 Expert** · [↑ API Gateway](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Basic] [You're asked to add a maintenance-mode page at the gateway for planned DB downtime; how do you serve a static response for write routes while keeping read routes live, and roll it back cleanly?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02882_You_re_asked_to_add_a_maintenance-mode_page_at_the_gateway_f.md>)
- [Deep] [Your gateway routes to services in two Kubernetes clusters via a multi-cluster setup; how do you handle failover between clusters when one's control plane is unreachable but its data plane still serves?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02796_Your_gateway_routes_to_services_in_two_Kubernetes_clusters_v.md>)
- [Deep] [How would you make the gateway resilient to a DNS resolver outage when it resolves upstream hostnames dynamically; what caching, negative-TTL handling, and stale-on-error behavior do you configure?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02875_How_would_you_make_the_gateway_resilient_to_a_DNS_resolver_o.md>)
- [Deep] [A mobile app on flaky networks retries aggressively through the gateway; how do you design gateway behavior (idempotency, dedupe, backpressure) to be resilient to unreliable clients?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02930_A_mobile_app_on_flaky_networks_retries_aggressively_through_.md>)
- [Deep] [You must decide the gateway's behavior when its own telemetry backend is down: does it drop metrics, buffer and risk memory, or block requests; adjudicate the failure mode, and design so losing observability never causes an availability incident yet you're not blind for long.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02769_You_must_decide_the_gateway_s_behavior_when_its_own_telemetr.md>)
- [Expert] [Your gateway is a single point of failure; walk through making it highly available with redundancy, health checks, and DNS/anycast failover, and where hidden SPOFs still hide.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02928_Your_gateway_is_a_single_point_of_failure_walk_through_makin.md>)
- [Expert] [A cascading failure started when the gateway's synchronous auth-service dependency slowed, backing up gateway threads and taking down all APIs; retrospect this and redesign the dependency so the gateway degrades gracefully rather than coupling every API's availability to one service.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02601_A_cascading_failure_started_when_the_gateway_s_synchronous_a.md>)
- [Expert] [Design the disaster-recovery runbook and architecture for a total control-plane loss: how do the data planes keep serving (fail static), how long can they run without config, and how do you rebuild the control plane from source of truth without a traffic blackout?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02647_Design_the_disaster-recovery_runbook_and_architecture_for_a_.md>)

## Related concepts

- [Org-wide API Standards](<../../API_Strategy/Org-wide_API_Standards.md>) *(API Strategy)*
- [Shared Config as Single Point of Failure](<../../../Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Shared_Config_as_Single_Point_of_Failure.md>) *(Distributed Systems Core)*
- [Backpressure and Load Shedding](<../../../System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*

