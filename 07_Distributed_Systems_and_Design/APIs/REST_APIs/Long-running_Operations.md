---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/Long-running_Operations"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Python"]
---

# Long-running Operations

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Intermediate] [Explain the request-response lifecycle when a browser calls a REST endpoint from JavaScript.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38383_Explain_the_request-response_lifecycle_when_a_browser_calls_.md>)
- [Intermediate] [What is the Retry-After header, and how could an LLM provider use it in a 429 response to tell your client when to try again?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38805_What_is_the_Retry-After_header_and_how_could_an_LLM_provider.md>)
- [Intermediate] [A downstream returns 200 quickly but the actual work is async and sometimes fails silently later. How do you design your integration to know the real outcome, using callbacks, reconciliation, or status polling?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38722_A_downstream_returns_200_quickly_but_the_actual_work_is_asyn.md>)
- [Deep] [You designed an endpoint returning a huge list without pagination and it timed out in production. How would you fix it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38441_You_designed_an_endpoint_returning_a_huge_list_without_pagin.md>)
- [Deep] [A search-as-you-type feature fires a REST call per keystroke, and out-of-order responses cause the wrong results to render. How do you handle cancellation and response ordering robustly in the client?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38593_A_search-as-you-type_feature_fires_a_REST_call_per_keystroke.md>)
- [Deep] [Your provider had a regional outage and every /chat call 503'd for 20 minutes. How would you design multi-provider failover at the REST layer so you route to a backup model without changing your endpoint's contract?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38866_Your_provider_had_a_regional_outage_and_every_chat_call_503_.md>)
- [Expert] [Your service calls a flaky downstream REST API and retries are amplifying its outage into your own. How do you distinguish a retry storm from real load, and what combination of circuit breaker, backoff, and load shedding do you apply?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38565_Your_service_calls_a_flaky_downstream_REST_API_and_retries_a.md>)
- [Expert] [Explain how you would move a hot write endpoint from synchronous DB write to write-behind through a queue while keeping the 201 Created contract and read-your-write semantics intact.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38649_Explain_how_you_would_move_a_hot_write_endpoint_from_synchro.md>)

## Related concepts

- [Serverless Limits and Throttling](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>) *(Cloud Fundamentals)*
- [Idempotent Order APIs](<../../System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [HttpClient and Resilient Outbound Calls](<../../../TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>) *(DotNet Platform)*

