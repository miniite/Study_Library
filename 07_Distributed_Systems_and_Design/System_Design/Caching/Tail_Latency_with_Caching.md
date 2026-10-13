---
id: "07_Distributed_Systems_and_Design/System_Design/Caching/Tail_Latency_with_Caching"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Caching"
tools: ["Redis"]
---

# Tail Latency with Caching

**L4 Advanced** · [↑ Caching](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Basic] [Write a memoized version of a recursive Fibonacci function and explain the speedup.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05724_Write_a_memoized_version_of_a_recursive_Fibonacci_function_a.md>)
- [Intermediate] [Explain how you would size a cache: given a request rate, average value size, and target hit ratio, how do you estimate the memory you need?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05450_Explain_how_you_would_size_a_cache_given_a_request_rate_aver.md>)
- [Deep] [What is the working set of an application, and why does it matter when sizing a cache?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05304_What_is_the_working_set_of_an_application_and_why_does_it_ma.md>)
- [Deep] [Explain how you would measure and control the tail-latency impact of cache serialization/deserialization when values are large protobufs versus JSON, and when you would switch formats.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05535_Explain_how_you_would_measure_and_control_the_tail-latency_i.md>)
- [Deep] [How would you cap the memory a single tenant can consume in a shared multi-tenant cache and evict fairly so a noisy tenant cannot starve others?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05557_How_would_you_cap_the_memory_a_single_tenant_can_consume_in_.md>)
- [Deep] [How would you cache the result of a rate-limit decision itself so the limiter is fast, without letting a cached 'allowed' verdict let a burst slip past the real limit?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05612_How_would_you_cache_the_result_of_a_rate-limit_decision_itse.md>)
- [Deep] [How would you estimate the memory a cache needs given request rate, value size distribution, and a target hit ratio, and how do you validate the estimate against production reality?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05632_How_would_you_estimate_the_memory_a_cache_needs_given_reques.md>)
- [Expert] [Your cache eviction rate is climbing while hit ratio drops. How do you tell whether you have a memory-sizing problem, a TTL problem, or a key-explosion problem?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05443_Your_cache_eviction_rate_is_climbing_while_hit_ratio_drops_H.md>)

## Related concepts

- [LRU Cache Design](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>) *(Data Structures and Algorithms)*
- [Cache Invalidation Across Regions](<../../Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>) *(Distributed Systems Core)*
- [Distributed Cache Design](<../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

