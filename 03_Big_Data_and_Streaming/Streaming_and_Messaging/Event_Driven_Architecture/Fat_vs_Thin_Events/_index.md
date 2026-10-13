---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture"
tools: ["Apache_Kafka"]
---

# Fat vs Thin Events

**L4 Advanced** · [↑ Event Driven Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Idempotent Consumers](<Idempotent_Consumers.md>)
- [Breaking Schema Changes](<Breaking_Schema_Changes.md>)

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Basic] [Why should events be named in the past tense, like OrderPlaced instead of PlaceOrder?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32602_Why_should_events_be_named_in_the_past_tense_like_OrderPlace.md>)
- [Intermediate] [How would you decide the granularity of an event, one big event versus several fine-grained ones?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32603_How_would_you_decide_the_granularity_of_an_event_one_big_eve.md>)
- [Intermediate] [What is a windowed aggregation in stream processing, and how do tumbling and sliding windows differ?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32685_What_is_a_windowed_aggregation_in_stream_processing_and_how_.md>)
- [Deep] [What information should a well-designed event payload contain, and should it carry full data or just an ID?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32560_What_information_should_a_well-designed_event_payload_contai.md>)
- [Deep] [Why does removing a required field from an event break forward compatibility, and how do you migrate safely?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32668_Why_does_removing_a_required_field_from_an_event_break_forwa.md>)
- [Deep] [Explain how you'd guarantee that a compacted topic never loses the latest state for a key even though older records are deleted, and what config bugs could still lose the latest.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32749_Explain_how_you_d_guarantee_that_a_compacted_topic_never_los.md>)
- [Deep] [You're asked whether to standardize the org on a single broker or run purpose-fit brokers per workload (log for streaming, queue for work-distribution, pub/sub for notifications). Defend a position and describe the operational tax of the polyglot choice.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32823_You_re_asked_whether_to_standardize_the_org_on_a_single_brok.md>)
- [Expert] [What is the outbox pattern's polling versus change-data-capture (CDC) approach, and what are the trade-offs?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32653_What_is_the_outbox_pattern_s_polling_versus_change-data-capt.md>)

## Related concepts

- [Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>) *(Distributed Systems Core)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Defining and Managing Messages in Log-Centric Architectures
- *Azure Adventures with C#* — Michal Switalik — event Grid
- *System Design Big Archive (2023)* — ByteByteGo — Differences in Event SOurcing System Design

