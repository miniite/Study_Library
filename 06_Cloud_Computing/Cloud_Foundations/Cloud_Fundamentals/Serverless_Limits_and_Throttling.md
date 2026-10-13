---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling"
level: 4
type: "concept"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: ["AWS", "Redis", "Python"]
---

# Serverless Limits and Throttling

**L4 Advanced** · [↑ Cloud Fundamentals](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Redis](<../../../TOOLS/Redis/_index.md>)
- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is exponential backoff and why is it useful when an LLM API returns a 429 too-many-requests error?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08379_What_is_exponential_backoff_and_why_is_it_useful_when_an_LLM.md>)
- [Intermediate] [What is a Lambda or serverless function's timeout limit and how could it break a long RAG generation request?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08445_What_is_a_Lambda_or_serverless_function_s_timeout_limit_and_.md>)
- [Intermediate] [How do you decide between a layer-4 and a layer-7 load balancer for a service that needs path-based routing but also handles long-lived WebSocket connections?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09149_How_do_you_decide_between_a_layer-4_and_a_layer-7_load_balan.md>)
- [Deep] [What is rate limiting and why is it important for a public cloud API?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08824_What_is_rate_limiting_and_why_is_it_important_for_a_public_c.md>)
- [Deep] [Explain how you would design connection pooling for a high-QPS inference service hitting a relational feature database so you do not exhaust connections during a spike.](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08218_Explain_how_you_would_design_connection_pooling_for_a_high-Q.md>)
- [Deep] [How would you detect and mitigate a slow-loris style attack against your API tier that keeps connections open without sending data?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08639_How_would_you_detect_and_mitigate_a_slow-loris_style_attack_.md>)
- [Deep] [Explain how you'd achieve zero-downtime cutover of a public DNS name from an old load balancer to a new one, accounting for DNS TTL and in-flight connections.](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08688_Explain_how_you_d_achieve_zero-downtime_cutover_of_a_public_.md>)
- [Deep] [Your service occasionally sees requests hang for exactly the socket timeout because a downstream half-closes the connection. How do you detect and defend against silent connection drops in the cloud?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08740_Your_service_occasionally_sees_requests_hang_for_exactly_the.md>)

## Related concepts

- [Org-wide Backpressure](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [API Rate Limiting](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*
- [Distributed Rate Limiter Design](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>) *(High Level Design)*

