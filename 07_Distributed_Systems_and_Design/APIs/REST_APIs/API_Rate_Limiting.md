---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Redis", "Python"]
---

# API Rate Limiting

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)
- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Intermediate] [What is rate limiting, and why do public REST APIs implement it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38373_What_is_rate_limiting_and_why_do_public_REST_APIs_implement_.md>)
- [Deep] [How would you protect a REST API from being overwhelmed by a single abusive client?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38436_How_would_you_protect_a_REST_API_from_being_overwhelmed_by_a.md>)
- [Deep] [How would you design an API rate-limit budget shared across a token bucket for normal calls and a separate stricter bucket for expensive endpoints, and how do you communicate both limits to a client cleanly?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38654_How_would_you_design_an_API_rate-limit_budget_shared_across_.md>)
- [Deep] [Your service's REST calls to a dependency use a shared connection pool, and one endpoint's slowness starves all others. How do you implement bulkheads so each downstream gets isolated resources?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38716_Your_service_s_REST_calls_to_a_dependency_use_a_shared_conne.md>)
- [Deep] [You want a canary rollout of a new model version behind the same /generate endpoint, routing 5% of traffic to it. How do you design the routing, response tagging, and metric split so you can compare quality and latency safely?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38881_You_want_a_canary_rollout_of_a_new_model_version_behind_the_.md>)
- [Expert] [Compare fixed-window, sliding-window, and token-bucket rate limiting in terms of burst handling and fairness.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38502_Compare_fixed-window_sliding-window_and_token-bucket_rate_li.md>)
- [Expert] [A single tenant's traffic spike is degrading the API for everyone. How do you implement fair-use rate limiting and per-tenant quotas without a central bottleneck, and how do you communicate limits to clients?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38611_A_single_tenant_s_traffic_spike_is_degrading_the_API_for_eve.md>)
- [Expert] [A REST endpoint must enforce that a user can perform an action at most N times per day across a distributed cluster, counting accurately even under concurrent requests and node failures. How do you build this quota?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38718_A_REST_endpoint_must_enforce_that_a_user_can_perform_an_acti.md>)

## Related concepts

- [Rate Limiting at the Gateway](<../API_Gateway/Rate_Limiting_at_the_Gateway.md>) *(API Gateway)*
- [Serverless Limits and Throttling](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>) *(Cloud Fundamentals)*
- [Distributed Rate Limiter Design](<../../System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>) *(High Level Design)*

