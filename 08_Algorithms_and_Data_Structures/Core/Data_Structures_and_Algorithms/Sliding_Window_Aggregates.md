---
id: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Aggregates"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms"
tools: []
---

# Sliding Window Aggregates

**L4 Advanced** · [↑ Data Structures and Algorithms](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you implement early stopping with patience, and what small data structure tracks the best-so-far metric and counter?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15498_How_would_you_implement_early_stopping_with_patience_and_wha.md>)
- [Intermediate] [How would you maintain a fixed-size buffer of the last 100 latency measurements from LLM calls to compute a rolling average, and which structure gives O(1) push and pop?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15698_How_would_you_maintain_a_fixed-size_buffer_of_the_last_100_l.md>)
- [Deep] [How would you design a data structure to answer 'sum of the last N samples' and 'sum in the last T seconds' for a metrics agent, and why does one need time-bucketing over a plain ring buffer?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15307_How_would_you_design_a_data_structure_to_answer_sum_of_the_l.md>)
- [Deep] [A notification digest job scans every user's followed entities nightly and takes 12 hours; how would you invert it with a per-entity subscriber index so work scales with what actually changed?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15377_A_notification_digest_job_scans_every_user_s_followed_entiti.md>)
- [Deep] [An online feature aggregation needs both 'sum of last N events' and 'sum over last T seconds' per entity; why does the time-window version require bucketing over a plain ring buffer, and how do you bound bucket count?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15537_An_online_feature_aggregation_needs_both_sum_of_last_N_event.md>)
- [Deep] [A drift-detection service keeps a sliding p99 of feature values cheaply; why is exact percentile over a window expensive and what sliding-window sketch or bucketed histogram bounds error and memory?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15580_A_drift-detection_service_keeps_a_sliding_p99_of_feature_val.md>)
- [Deep] [Your model store must answer 'sum of prediction amounts for entity X between time A and B' over a billion-row append-only log; how do time-bucketed prefix sums or a Fenwick-over-buckets beat scanning raw rows?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15612_Your_model_store_must_answer_sum_of_prediction_amounts_for_e.md>)
- [Expert] [Design a data structure for autoscaling decisions that keeps a sliding p95 of request latency cheaply; why is exact percentile over a window expensive and what approximation would you accept?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15310_Design_a_data_structure_for_autoscaling_decisions_that_keeps.md>)

## Related concepts

- [Clickstream Counting Pipeline](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*
- [Sorting Cost in Services](<../Complexity_Analysis/Sorting_Cost_in_Services.md>) *(Complexity Analysis)*
- [Time-based Windows](<../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>) *(Window Functions)*

