---
id: "07_Distributed_Systems_and_Design/APIs/API_Gateway/Rate_Limiting_at_the_Gateway"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/API_Gateway"
tools: ["Redis"]
---

# Rate Limiting at the Gateway

**L4 Advanced** · [↑ API Gateway](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Describe how you'd wire per-route rate limiting in Kong backed by Redis across three replicas; what happens to limits when a Kong node loses Redis, and how do you decide the local-vs-cluster policy?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02792_Describe_how_you_d_wire_per-route_rate_limiting_in_Kong_back.md>)
- [Intermediate] [How would you build a gateway that enforces different max-request-body limits per route (1MB for JSON APIs, 500MB for uploads) while streaming so a large upload never buffers fully in gateway memory?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02461_How_would_you_build_a_gateway_that_enforces_different_max-re.md>)
- [Deep] [How do you decide the gateway's upstream connection pool size per service given a fixed number of backend pods, and what symptom tells you the pool is too small (queuing) vs too large (backend overload)?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02821_How_do_you_decide_the_gateway_s_upstream_connection_pool_siz.md>)
- [Deep] [Your token-bucket rate limiter in Redis uses GET then SET and two concurrent requests both pass when only one should; how do you make the limiter atomic, and what Lua/eval or CRDT approach do you use?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02453_Your_token-bucket_rate_limiter_in_Redis_uses_GET_then_SET_an.md>)
- [Deep] [How do you design a gateway that enforces a global maximum concurrent-connection limit to protect an origin database that can only handle N connections, and how does this differ from per-second rate limiting?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02541_How_do_you_design_a_gateway_that_enforces_a_global_maximum_c.md>)
- [Deep] [Your gateway sits in front of a GraphQL server and REST services; how do you route, rate-limit, and secure GraphQL differently given queries can be arbitrarily expensive?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02896_Your_gateway_sits_in_front_of_a_GraphQL_server_and_REST_serv.md>)
- [Deep] [Your gateway enforces a global concurrent-connection cap to protect a fragile shared database, but this couples unrelated services' availability; adjudicate whether protecting the database at the gateway is the right layer, or whether the fix belongs deeper in the architecture.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02625_Your_gateway_enforces_a_global_concurrent-connection_cap_to_.md>)
- [Expert] [Your sliding-window rate limiter stores a sorted set of timestamps per key and memory blew up for a hot key with millions of requests; how do you cap memory while keeping accuracy acceptable, and when is an approximate counter fine?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02457_Your_sliding-window_rate_limiter_stores_a_sorted_set_of_time.md>)

## Related concepts

- [API Rate Limiting](<../REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*
- [Distributed Rate Limiter Design](<../../System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>) *(High Level Design)*
- [Serverless Limits and Throttling](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>) *(Cloud Fundamentals)*

