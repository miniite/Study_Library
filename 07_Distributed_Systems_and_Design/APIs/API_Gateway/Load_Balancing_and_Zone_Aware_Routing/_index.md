---
id: "07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/API_Gateway"
tools: ["AWS", "Kubernetes"]
---

# Load Balancing and Zone Aware Routing

**L4 Advanced** · [↑ API Gateway](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Gateway Cost Metering and Budgets](<Gateway_Cost_Metering_and_Budgets.md>)

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [How would you instrument a synthetic canary that probes each critical gateway route every minute from outside your VPC, and how do you keep those probes from double-counting in your real availability SLO?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02812_How_would_you_instrument_a_synthetic_canary_that_probes_each.md>)
- [Intermediate] [Explain how you'd implement weighted least-connections load balancing at the gateway and why it can beat round-robin when upstream instances have heterogeneous capacity or in-flight long requests.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02495_Explain_how_you_d_implement_weighted_least-connections_load_.md>)
- [Deep] [You want blue-green at the gateway layer where you flip an entire environment; how do you structure DNS, weighted target groups, and connection draining so the cutover and rollback are both instant and safe?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02798_You_want_blue-green_at_the_gateway_layer_where_you_flip_an_e.md>)
- [Deep] [How would you run the gateway across three AZs so the loss of one AZ doesn't drop more than a third of capacity, and what pre-provisioned headroom keeps the remaining two from tipping over?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02835_How_would_you_run_the_gateway_across_three_AZs_so_the_loss_o.md>)
- [Deep] [Explain how you'd implement priority-based load shedding at the gateway so that during overload you drop low-value traffic (health-check pollers, analytics) before checkout requests; how do you classify and tag priority?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02452_Explain_how_you_d_implement_priority-based_load_shedding_at_.md>)
- [Deep] [How would you implement a canary that shifts traffic by percentage AND by cohort (e.g., 5% + all internal employees) at the gateway, and how do you make the routing decision sticky per user across the canary window?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02523_How_would_you_implement_a_canary_that_shifts_traffic_by_perc.md>)
- [Deep] [How would you implement geo-based or latency-based routing at the gateway/edge so users hit the nearest region; what consistency and session-affinity issues arise?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02897_How_would_you_implement_geo-based_or_latency-based_routing_a.md>)
- [Deep] [Explain how HTTP/2 and gRPC change gateway design vs HTTP/1.1; how do you proxy gRPC, handle multiplexing, and deal with load balancing long-lived connections?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02910_Explain_how_HTTP_2_and_gRPC_change_gateway_design_vs_HTTP_1_.md>)

## Related concepts

- [Serverless Limits and Throttling](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>) *(Cloud Fundamentals)*
- [API Rate Limiting](<../../REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*
- [DNS Failover](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_Failover.md>) *(Networking)*

