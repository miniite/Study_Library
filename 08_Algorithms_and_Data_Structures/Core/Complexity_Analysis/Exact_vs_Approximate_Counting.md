---
id: "08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Exact_vs_Approximate_Counting"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Complexity_Analysis"
tools: []
---

# Exact vs Approximate Counting

**L4 Advanced** · [↑ Complexity Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [A recommendation endpoint does an O(n*m) cross-join in application code; describe how you'd profile it, prove the complexity is the bottleneck, and stage the fix without a big-bang rewrite.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43369_A_recommendation_endpoint_does_an_O_n_m_cross-join_in_applic.md>)
- [Intermediate] [A form with real-time validation runs an O(n) rule set on every keystroke across many fields; how do you reason about the aggregate complexity and where debounce, memo, and incremental validation each apply?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43441_A_form_with_real-time_validation_runs_an_O_n_rule_set_on_eve.md>)
- [Deep] [Why does a large OFFSET in SQL pagination degrade toward O(n), and what's the complexity-friendly alternative?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43331_Why_does_a_large_OFFSET_in_SQL_pagination_degrade_toward_O_n.md>)
- [Deep] [You're asked to add idempotency keys to prevent duplicate payments; analyze the time and space complexity of the dedup store and its retention/cleanup as a complexity concern.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43421_You_re_asked_to_add_idempotency_keys_to_prevent_duplicate_pa.md>)
- [Deep] [How do you reason about the complexity of validating uniqueness at scale (e.g., unique username) using a DB unique index (O(log n) via B-tree) versus an application-side in-memory set, including consistency risk?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43460_How_do_you_reason_about_the_complexity_of_validating_uniquen.md>)
- [Deep] [You're asked to keep a live counter of unique active users; how do you reason about exact O(n)-space sets versus HyperLogLog's near-constant space and justify the accuracy trade-off to product?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43501_You_re_asked_to_keep_a_live_counter_of_unique_active_users_h.md>)
- [Deep] [How do you reason about the complexity of choosing keyset (cursor) pagination over offset for a deep-scroll feed API, including how sort keys and deletes affect the O(log n) seek guarantee?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43528_How_do_you_reason_about_the_complexity_of_choosing_keyset_cu.md>)
- [Deep] [You must design a uniqueness check for a high-write feature (unique handle); how do you reason about a DB unique index's O(log n) B-tree enforcement versus an app-side in-memory set and its consistency risk?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Q-43554_You_must_design_a_uniqueness_check_for_a_high-write_feature_.md>)

## Related concepts

- [Stream Join Data Structures](<../Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Set Intersection Structures](<../Data_Structures_and_Algorithms/Set_Intersection_Structures.md>) *(Data Structures and Algorithms)*
- [Clickstream Counting Pipeline](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*

## References

- *Coding Interview Patterns* — Alex Xu, Shaun Gunawardane — t he c,urrent window, it ' s a dup l ica t e charac t er insupthsup e # wi ndo w. if (s (right in

