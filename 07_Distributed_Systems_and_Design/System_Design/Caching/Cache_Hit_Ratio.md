---
id: "07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Hit_Ratio"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Caching"
tools: ["Redis"]
---

# Cache Hit Ratio

**L4 Advanced** · [↑ Caching](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Basic] [Explain the difference between a cache hit and a cache miss.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05406_Explain_the_difference_between_a_cache_hit_and_a_cache_miss.md>)
- [Basic] [What is cache hit ratio and how would you measure it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05517_What_is_cache_hit_ratio_and_how_would_you_measure_it.md>)
- [Intermediate] [What is a cache-hit ratio of 95% telling you, and would raising it to 99% always be worth it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05308_What_is_a_cache-hit_ratio_of_95_telling_you_and_would_raisin.md>)
- [Intermediate] [What metrics (hit ratio, eviction rate, p99 latency, memory) would you graph on a cache dashboard and why?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05392_What_metrics_hit_ratio_eviction_rate_p99_latency_memory_woul.md>)
- [Intermediate] [You added a Redis cache but the hit ratio is very low. How would you debug this?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05595_You_added_a_Redis_cache_but_the_hit_ratio_is_very_low_How_wo.md>)
- [Intermediate] [A downstream team asks you to expose your cache hit ratio as an SLI. What exactly do you define as a hit, how do you avoid gaming the metric, and what SLO would you commit to?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05527_A_downstream_team_asks_you_to_expose_your_cache_hit_ratio_as.md>)
- [Intermediate] [You inherit a cache with a 40% hit ratio and are asked to improve it. Walk through your investigation, hypotheses, and the order in which you would test each lever.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05558_You_inherit_a_cache_with_a_40_hit_ratio_and_are_asked_to_imp.md>)
- [Deep] [Your cache hit ratio suddenly dropped from 95% to 60% at 2am. Walk me through your investigation.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05393_Your_cache_hit_ratio_suddenly_dropped_from_95_to_60_at_2am_W.md>)

## Related concepts

- [Cache Invalidation Across Regions](<../../Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>) *(Distributed Systems Core)*
- [LRU Cache Design](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>) *(Data Structures and Algorithms)*
- [Tail Latency After Expansion](<../System_Design_Core/Tail_Latency_After_Expansion.md>) *(System Design Core)*

