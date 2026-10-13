---
id: "TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls"
level: 4
type: "concept"
topic: "TOOLS/DotNet/DotNet_Platform"
tools: ["DotNet", "Redis"]
---

# HttpClient and Resilient Outbound Calls

**L4 Advanced** · [↑ DotNet Platform](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [DotNet](<../_index.md>)
- [Redis](<../../Redis/_index.md>)

## Practice questions

- [Intermediate] [How do you safely deprecate and remove an endpoint consumed by unknown internal clients — what telemetry, sunset headers, and rollout gating do you put in place first?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00163_How_do_you_safely_deprecate_and_remove_an_endpoint_consumed_.md>)
- [Deep] [What is socket exhaustion from misusing HttpClient and how does IHttpClientFactory prevent it?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00004_What_is_socket_exhaustion_from_misusing_HttpClient_and_how_d.md>)
- [Deep] [Explain how you'd design cache invalidation for a read-heavy .NET service using Redis — cache-aside vs write-through, stampede protection, and TTL vs event-driven invalidation trade-offs.](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00151_Explain_how_you_d_design_cache_invalidation_for_a_read-heavy.md>)
- [Deep] [Your integration tests are flaky because of shared state between the test host and a real DB — how would you structure WebApplicationFactory, test containers, and data isolation for reliable parallel tests?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00176_Your_integration_tests_are_flaky_because_of_shared_state_bet.md>)
- [Deep] [How would you implement output caching / response caching for a .NET API considering per-user variance, cache keys with Vary headers, and invalidation when underlying data changes?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00192_How_would_you_implement_output_caching_response_caching_for_.md>)
- [Deep] [How would you implement a health-check design that distinguishes liveness from readiness so Kubernetes doesn't kill a pod that's merely waiting on a slow dependency at startup?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00226_How_would_you_implement_a_health-check_design_that_distingui.md>)
- [Expert] [Design an idempotent payment-processing endpoint that survives duplicate webhook deliveries and client retries — what key do you dedupe on, where do you store it, and how do you handle the race between two concurrent duplicates?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00144_Design_an_idempotent_payment-processing_endpoint_that_surviv.md>)
- [Expert] [How do you design a saga/compensation flow across multiple .NET services for a booking that touches inventory, payment, and notification, and how do you keep it recoverable after a crash mid-flow?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00171_How_do_you_design_a_saga_compensation_flow_across_multiple_N.md>)

## Related concepts

- [Two-level Cache Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*
- [Progressive Results](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Progressive_Results.md>) *(REST APIs)*
- [Lock Granularity Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>) *(Low Level Design)*

