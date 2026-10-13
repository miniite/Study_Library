---
id: "07_Distributed_Systems_and_Design/System_Design/Caching/What_to_Cache"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Caching"
tools: ["Redis"]
---

# What to Cache

**L4 Advanced** · [↑ Caching](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Roughly how much faster is reading from RAM versus a spinning disk versus a network round-trip? Why does that motivate caching?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05303_Roughly_how_much_faster_is_reading_from_RAM_versus_a_spinnin.md>)
- [Intermediate] [Why did you choose to cache this particular data — how do you decide what is worth caching?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05606_Why_did_you_choose_to_cache_this_particular_data_how_do_you_.md>)
- [Intermediate] [How would you decide compression on/off for cached values, and how do you measure whether the CPU cost is worth the memory and network savings?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05475_How_would_you_decide_compression_on_off_for_cached_values_an.md>)
- [Deep] [When you A/B test a caching change, what would you measure to prove the cache actually helped rather than hurt?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05403_When_you_A_B_test_a_caching_change_what_would_you_measure_to.md>)
- [Deep] [When can caching actually hurt performance or add more complexity than it's worth?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05764_When_can_caching_actually_hurt_performance_or_add_more_compl.md>)
- [Deep] [Your caching layer added complexity but a review shows only a 3% latency improvement. How do you decide whether to keep it, tune it, or rip it out?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05478_Your_caching_layer_added_complexity_but_a_review_shows_only_.md>)
- [Deep] [How would you decide the cost/benefit of compressing cached values when your cache is network-bound versus CPU-bound, and how do you measure which bottleneck you actually have?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05590_How_would_you_decide_the_cost_benefit_of_compressing_cached_.md>)
- [Deep] [A skeptical staff engineer argues caching is premature optimization and you should measure first. In an architecture review, how do you distinguish workloads where caching should be designed in from the start versus added later?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05661_A_skeptical_staff_engineer_argues_caching_is_premature_optim.md>)

## Related concepts

- [Separating Compute and Storage](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Separating_Compute_and_Storage.md>) *(Cloud Fundamentals)*
- [Database Cost Optimization](<../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*
- [Distributed Cache Design](<../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

