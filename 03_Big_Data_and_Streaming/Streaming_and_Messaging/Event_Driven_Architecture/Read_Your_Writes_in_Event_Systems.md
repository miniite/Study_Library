---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Read_Your_Writes_in_Event_Systems"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture"
tools: ["Apache_Kafka"]
---

# Read Your Writes in Event Systems

**L4 Advanced** · [↑ Event Driven Architecture](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [How do you reset a consumer group's offset to reprocess from the beginning or from a timestamp?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32622_How_do_you_reset_a_consumer_group_s_offset_to_reprocess_from.md>)
- [Deep] [How would you replay events from the past to rebuild a downstream service's state?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32585_How_would_you_replay_events_from_the_past_to_rebuild_a_downs.md>)
- [Deep] [How would you prevent an infinite loop of events where a consumer publishes an event that eventually re-triggers itself?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32594_How_would_you_prevent_an_infinite_loop_of_events_where_a_con.md>)
- [Deep] [How do you keep a saga idempotent so a retried step doesn't double-book or double-refund?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32660_How_do_you_keep_a_saga_idempotent_so_a_retried_step_doesn_t_.md>)
- [Deep] [Explain how you'd detect that a consumer is silently committing offsets for messages it never actually processed (a code path swallows exceptions). What guardrails prevent this class of bug?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32710_Explain_how_you_d_detect_that_a_consumer_is_silently_committ.md>)
- [Deep] [Explain how you'd detect and break an event feedback loop where service A's event causes B to emit an event that A consumes and re-emits, under partial deploys where only some instances have the fix.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32763_Explain_how_you_d_detect_and_break_an_event_feedback_loop_wh.md>)
- [Deep] [You've chosen eventual consistency for this read model. Product now says a specific screen needs read-your-writes. Do you compromise the whole architecture, add a synchronous fast-path, or push back on the requirement? Defend your choice and its blast radius.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32865_You_ve_chosen_eventual_consistency_for_this_read_model_Produ.md>)
- [Expert] [How would you rebuild a broken read model from the event log while the system keeps serving traffic?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32664_How_would_you_rebuild_a_broken_read_model_from_the_event_log.md>)

## Related concepts

- [Compensating Transactions](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>) *(Low Level Design)*
- [Idempotent Order APIs](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [Two-level Cache Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

