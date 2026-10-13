---
id: "07_Distributed_Systems_and_Design/APIs/API_Gateway/Timeouts_Retries_and_Backoff"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/API_Gateway"
tools: []
---

# Timeouts Retries and Backoff

**L4 Advanced** · [↑ API Gateway](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Your gateway runs on spot instances to cut cost and a spot reclamation drained a node mid-request; how do you handle spot interruptions gracefully for a latency-sensitive gateway, and when is spot a bad idea here?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02826_Your_gateway_runs_on_spot_instances_to_cut_cost_and_a_spot_r.md>)
- [Intermediate] [How do you decide gateway retry policy so idempotent GETs retry with backoff and jitter but POSTs don't, and how do you enforce this per-route without a team accidentally retrying a payment write?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02856_How_do_you_decide_gateway_retry_policy_so_idempotent_GETs_re.md>)
- [Intermediate] [How do you design gateway behavior when an upstream returns Retry-After on a 429/503; should the gateway honor it, transform it, or hide it from clients, and how does that interact with your own retry policy?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02458_How_do_you_design_gateway_behavior_when_an_upstream_returns_.md>)
- [Deep] [How do you decide the gateway's idle vs total connection timeouts to defend against slowloris-style slow clients without cutting off a legitimately slow report download; how do these timeouts interact?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02887_How_do_you_decide_the_gateway_s_idle_vs_total_connection_tim.md>)
- [Deep] [Your gateway emits one span per request but downstream fan-out spans aren't linked because trace context is dropped on async retries; how do you ensure trace propagation survives retries, queuing, and thread hops?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02475_Your_gateway_emits_one_span_per_request_but_downstream_fan-o.md>)
- [Deep] [A gateway must apply different timeout and retry policies for reads vs writes on the same upstream; how do you classify request safety at the edge and avoid retrying a write that may have partially applied?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02547_A_gateway_must_apply_different_timeout_and_retry_policies_fo.md>)
- [Expert] [How do you propagate a deadline (not just a timeout) through the gateway so that if a client has 200ms left, each downstream hop shrinks its own budget accordingly instead of every hop starting a fresh timeout?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02449_How_do_you_propagate_a_deadline_not_just_a_timeout_through_t.md>)
- [Expert] [How do you design the gateway's behavior when an upstream keeps a connection open but sends bytes very slowly (slowloris-style, or a genuinely slow report); how do idle vs total timeouts and read deadlines differ and interact?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02494_How_do_you_design_the_gateway_s_behavior_when_an_upstream_ke.md>)

## Related concepts

- [Long-running Operations](<../REST_APIs/Long-running_Operations.md>) *(REST APIs)*
- [Serverless Limits and Throttling](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>) *(Cloud Fundamentals)*
- [API Rate Limiting](<../REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*

