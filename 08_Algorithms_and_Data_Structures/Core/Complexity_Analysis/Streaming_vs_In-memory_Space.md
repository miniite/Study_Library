---
id: "08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Streaming_vs_In-memory_Space"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Complexity_Analysis"
tools: []
---

# Streaming vs In-memory Space

**L4 Advanced** · [↑ Complexity Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [Why is an in-place array reversal O(1) space while building a new reversed array is O(n) space?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43351_Why_is_an_in-place_array_reversal_O_1_space_while_building_a.md>)
- [Intermediate] [Why can an unmemoized O(n) computation in a render function become effectively O(n * renders), and how does useMemo help?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43334_Why_can_an_unmemoized_O_n_computation_in_a_render_function_b.md>)
- [Intermediate] [How do you reason about the complexity impact of choosing an unbounded cache versus an LRU/TTL cache, both for latency and for memory (space) growth over time in prod?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43391_How_do_you_reason_about_the_complexity_impact_of_choosing_an.md>)
- [Intermediate] [A client caches API results in an ever-growing JS Map for "instant" repeat access; how do you reason about the browser-tab memory complexity over a long session and how you'd bound it?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43459_A_client_caches_API_results_in_an_ever-growing_JS_Map_for_in.md>)
- [Intermediate] [An export endpoint materializes millions of rows into a list before writing the file, OOMing at scale; how do you reason about O(n) space versus a streaming cursor, and where's the safe cutover point?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43510_An_export_endpoint_materializes_millions_of_rows_into_a_list.md>)
- [Deep] [Your batch job is O(n) but consumes O(n) memory and OOMs at scale; how would you redesign it to bounded/constant memory while keeping throughput acceptable?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43364_Your_batch_job_is_O_n_but_consumes_O_n_memory_and_OOMs_at_sc.md>)
- [Deep] [A service's memory grows unbounded over days; how do you distinguish a genuine O(n)-in-input space complexity issue from a leak, and what tools/metrics tell them apart?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43447_A_service_s_memory_grows_unbounded_over_days_how_do_you_dist.md>)
- [Deep] [How do you reason about the complexity of a permission check cached per (user, resource) pair, where the cache is O(users x resources) in the worst case, and how you'd bound it with scoping and TTL?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43538_How_do_you_reason_about_the_complexity_of_a_permission_check.md>)

## Related concepts

- [Stream Join Data Structures](<../Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Tail Latency with Caching](<../../../07_Distributed_Systems_and_Design/System_Design/Caching/Tail_Latency_with_Caching.md>) *(Caching)*
- [Idempotent Consumer Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>) *(Low Level Design)*

