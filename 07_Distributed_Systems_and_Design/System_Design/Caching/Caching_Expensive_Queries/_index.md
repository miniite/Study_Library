---
id: "07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Caching"
tools: ["Redis"]
---

# Caching Expensive Queries

**L4 Advanced** · [↑ Caching](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Multi-layer Caching Architecture](<Multi-layer_Caching_Architecture.md>)

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [How would you cache the output of an expensive aggregation query that changes only a few times per day?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05364_How_would_you_cache_the_output_of_an_expensive_aggregation_q.md>)
- [Deep] [How would you cache negative results (a 'user not found') without letting an attacker flood you with junk keys?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05375_How_would_you_cache_negative_results_a_user_not_found_withou.md>)
- [Deep] [How would you design a cache-warming pipeline that runs continuously in the background so the cache is never cold, and how do you keep it from overwhelming the DB?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05417_How_would_you_design_a_cache-warming_pipeline_that_runs_cont.md>)
- [Deep] [How would you cache a value that a handful of admin users can change instantly but millions of end users read, so admins see writes immediately and users within seconds?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05482_How_would_you_cache_a_value_that_a_handful_of_admin_users_ca.md>)
- [Deep] [Explain how you would design negative caching for a 'resource not found' response so an attacker cannot flood you with junk keys and exhaust memory.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05508_Explain_how_you_would_design_negative_caching_for_a_resource.md>)
- [Deep] [You must cache responses for an endpoint that returns different data based on 12 query parameters, most of which are irrelevant to the result. How do you avoid a combinatorial key explosion?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05571_You_must_cache_responses_for_an_endpoint_that_returns_differ.md>)
- [Deep] [How would you size and shard a cache for a workload where 5% of keys account for 95% of reads, so the hot keys never contend and the cold tail does not waste memory?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05610_How_would_you_size_and_shard_a_cache_for_a_workload_where_5_.md>)
- [Expert] [How does adding or removing a node affect key distribution in a sharded cache?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05737_How_does_adding_or_removing_a_node_affect_key_distribution_i.md>)

## Related concepts

- [LRU Cache Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>) *(Data Structures and Algorithms)*
- [Distributed Cache Design](<../../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [Read-through and Write-behind Caching](<../../System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>) *(System Design Core)*

## References

- *System Design: The Big Archive* — Alex Xu (ByteByteGo) — Cache miss attack

