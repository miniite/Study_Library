---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Idempotent_Consumers"
level: 5
type: "deep-dive"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture"
tools: ["Apache_Kafka", "Redis"]
---

# Idempotent Consumers

**L5 Expert** · [↑ Event Driven Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Explain at-least-once, at-most-once, and exactly-once delivery semantics.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32502_Explain_at-least-once_at-most-once_and_exactly-once_delivery.md>)
- [Deep] [What happens if two consumers accidentally process the same message? How would you detect and prevent it?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32582_What_happens_if_two_consumers_accidentally_process_the_same_.md>)
- [Deep] [Design the topic layout and keying for a ride-hailing system where per-driver location updates must stay ordered but you have 500k active drivers. Justify partition count and key choice.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32704_Design_the_topic_layout_and_keying_for_a_ride-hailing_system.md>)
- [Deep] [A consumer must fan out one incoming event into per-recipient work for up to 5 million recipients. How do you avoid a single giant transaction and keep the fan-out restartable?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32762_A_consumer_must_fan_out_one_incoming_event_into_per-recipien.md>)
- [Deep] [Your proposed design adds a queue between two services 'for resilience.' The interviewer asks: haven't you just moved the failure to the queue and added latency, ordering, and dedup problems you didn't have before? Defend when the trade is net-positive and when it isn't.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32874_Your_proposed_design_adds_a_queue_between_two_services_for_r.md>)
- [Expert] [How would you architect a fan-in aggregation where events from 200 shards must be joined into a per-customer view, tolerating a shard being hours behind the others?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32714_How_would_you_architect_a_fan-in_aggregation_where_events_fr.md>)
- [Expert] [Explain how you'd guarantee ordering within a consumer that processes a single partition using a thread pool for parallelism. What's the mechanism that preserves per-key order?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32793_Explain_how_you_d_guarantee_ordering_within_a_consumer_that_.md>)
- [Expert] [Architect the event system for a payments platform where every event must be exactly-once against the ledger, auditable for a decade, and resilient to a full region loss. Walk me through the layers and where you accept eventual consistency versus demand strong guarantees.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32847_Architect_the_event_system_for_a_payments_platform_where_eve.md>)

## Related concepts

- [Dedup Contracts](<../../Delivery_Semantics/Dedup_Contracts.md>) *(Delivery Semantics)*
- [Idempotent Order APIs](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>) *(High Level Design)*

