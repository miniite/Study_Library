---
id: "07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Stampede_Prevention"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Caching"
tools: []
---

# Cache Stampede Prevention

**L4 Advanced** · [↑ Caching](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is FIFO cache eviction and when might it be a bad choice?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05429_What_is_FIFO_cache_eviction_and_when_might_it_be_a_bad_choic.md>)
- [Intermediate] [Your cache uses TTL-based expiry but you observe correlated expiries causing periodic DB spikes. How do you diagnose that pattern and apply jitter without losing freshness guarantees?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05569_Your_cache_uses_TTL-based_expiry_but_you_observe_correlated_.md>)
- [Deep] [How would you implement per-request locking or a mutex so only one caller repopulates a hot key on expiry?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05322_How_would_you_implement_per-request_locking_or_a_mutex_so_on.md>)
- [Deep] [What is 'request coalescing' (single-flight) and how does it protect the database during a stampede?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05324_What_is_request_coalescing_single-flight_and_how_does_it_pro.md>)
- [Deep] [What is a cache stampede (thundering herd) and how would you prevent it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05551_What_is_a_cache_stampede_thundering_herd_and_how_would_you_p.md>)
- [Deep] [What is cache avalanche and how can staggered TTLs help mitigate it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05573_What_is_cache_avalanche_and_how_can_staggered_TTLs_help_miti.md>)
- [Deep] [Describe how you would introduce request coalescing at the service layer and prove it reduced DB load during a stampede in staging.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05442_Describe_how_you_would_introduce_request_coalescing_at_the_s.md>)
- [Deep] [Explain how you would use TTL jitter and staggered expiries to prevent thousands of keys expiring at the same second and stampeding the DB.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05457_Explain_how_you_would_use_TTL_jitter_and_staggered_expiries_.md>)

## Related concepts

- [Cache Invalidation Across Regions](<../../Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>) *(Distributed Systems Core)*
- [Distributed Cache Design](<../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [Two-level Cache Design](<../Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

