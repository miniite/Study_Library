---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture"
tools: ["Apache_Kafka"]
---

# Lag-based Consumer Autoscaling

**L4 Advanced** · [↑ Event Driven Architecture](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [What is consumer lag and how do you monitor it?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32538_What_is_consumer_lag_and_how_do_you_monitor_it.md>)
- [Intermediate] [How do you decide whether an operation belongs in the synchronous request path or should be pushed to a queue? Give the concrete criteria you'd apply during design review.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32730_How_do_you_decide_whether_an_operation_belongs_in_the_synchr.md>)
- [Deep] [A consumer is processing messages slowly and lag keeps growing. Walk me through how you would investigate.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32552_A_consumer_is_processing_messages_slowly_and_lag_keeps_growi.md>)
- [Deep] [A user updates their profile but the search index shows old data for a few seconds. How do you explain and handle this?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32589_A_user_updates_their_profile_but_the_search_index_shows_old_.md>)
- [Deep] [How would you design a thundering-herd-safe fan-out where one event triggers work for a million users?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32688_How_would_you_design_a_thundering-herd-safe_fan-out_where_on.md>)
- [Deep] [Explain how you'd cap the tail latency (p99) of an async pipeline where most messages are fast but 1% call a slow external API. How do you keep the fast ones from queueing behind slow ones?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32729_Explain_how_you_d_cap_the_tail_latency_p99_of_an_async_pipel.md>)
- [Deep] [How would you monitor for and alert on a producer that has silently stopped emitting a low-volume-but-critical event (e.g. fraud-flagged), where lag metrics won't catch it?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32756_How_would_you_monitor_for_and_alert_on_a_producer_that_has_s.md>)
- [Deep] [During a broker upgrade, consumers on the old client version can't read messages produced with a new compression codec. How do you sequence the rollout to avoid this?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32796_During_a_broker_upgrade_consumers_on_the_old_client_version_.md>)

## Related concepts

- [Fan-out Trade-offs](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Org-wide Backpressure](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Sync vs Async Processing](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Sync_vs_Async_Processing.md>) *(Cloud Fundamentals)*

