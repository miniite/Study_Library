---
id: "07_Distributed_Systems_and_Design/System_Design/Caching/Write-through_and_Write-back_Consistency"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Caching"
tools: ["Redis"]
---

# Write-through and Write-back Consistency

**L4 Advanced** · [↑ Caching](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Explain the write-through caching strategy and its trade-offs.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05473_Explain_the_write-through_caching_strategy_and_its_trade-off.md>)
- [Deep] [Explain the read-your-own-writes problem with caching: a user updates data but a cached read shows the old value.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05398_Explain_the_read-your-own-writes_problem_with_caching_a_user.md>)
- [Deep] [What is write-around caching and when would you use it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05495_What_is_write-around_caching_and_when_would_you_use_it.md>)
- [Deep] [How do you handle concurrent reads and writes safely in an in-memory cache?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05760_How_do_you_handle_concurrent_reads_and_writes_safely_in_an_i.md>)
- [Deep] [How do you reason about the consistency guarantees your cache actually provides end-to-end, and how would you communicate acceptable staleness to product and legal?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05505_How_do_you_reason_about_the_consistency_guarantees_your_cach.md>)
- [Deep] [You must influence a peer principal who insists on write-through everywhere for correctness, at a real latency and cost penalty. How do you change their mind without authority, using data and shared principles?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05656_You_must_influence_a_peer_principal_who_insists_on_write-thr.md>)
- [Expert] [How do you keep a two-level near-cache-plus-remote-cache design coherent when one app instance's local copy goes stale after another instance writes?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05456_How_do_you_keep_a_two-level_near-cache-plus-remote-cache_des.md>)
- [Expert] [Explain how you would cache the result of a distributed transaction so readers see the committed value only after all participants commit, never a partially-visible intermediate state.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05556_Explain_how_you_would_cache_the_result_of_a_distributed_tran.md>)

## Related concepts

- [Read-through and Write-behind Caching](<../System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>) *(System Design Core)*
- [Distributed Cache Design](<../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [Read Your Writes with Replicas](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*

