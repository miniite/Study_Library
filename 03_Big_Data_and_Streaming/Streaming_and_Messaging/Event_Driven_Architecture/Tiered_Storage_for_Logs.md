---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture"
tools: ["Apache_Kafka", "AWS"]
---

# Tiered Storage for Logs

**L4 Advanced** · [↑ Event Driven Architecture](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [What is the difference between Kafka and RabbitMQ, and when would you pick one over the other?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32510_What_is_the_difference_between_Kafka_and_RabbitMQ_and_when_w.md>)
- [Deep] [How does Kafka differ from a traditional queue in how it retains and replays messages?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32519_How_does_Kafka_differ_from_a_traditional_queue_in_how_it_ret.md>)
- [Deep] [What is min.insync.replicas and how does it interact with acks=all to prevent data loss?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32634_What_is_min_insync_replicas_and_how_does_it_interact_with_ac.md>)
- [Deep] [You have a Kafka topic where each message is ~2MB (embedded document blobs). Consumers keep hitting rebalances and OOMs. How would you redesign the payload and consumer config to fix this?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32698_You_have_a_Kafka_topic_where_each_message_is_2MB_embedded_do.md>)
- [Deep] [When would you push transformation/enrichment into Kafka Streams / Flink versus keeping plain consumers, purely from an operational-complexity and correctness standpoint?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32766_When_would_you_push_transformation_enrichment_into_Kafka_Str.md>)
- [Deep] [An interviewer asks you to defend running your own Kafka over managed. Six months later the pager burden is crushing your platform team. At what signal do you reverse the decision, and how do you migrate to managed without admitting the original call was wrong in a way that loses the team's trust?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32873_An_interviewer_asks_you_to_defend_running_your_own_Kafka_ove.md>)
- [Expert] [Explain how you'd bound the blast radius when one misbehaving producer floods a shared Kafka cluster and starves other tenants. What isolation and quota controls exist?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32701_Explain_how_you_d_bound_the_blast_radius_when_one_misbehavin.md>)
- [Expert] [You're setting the org's messaging strategy for the next three years across 40 teams. How do you decide between one central paved-road Kafka platform, a mesh of per-domain brokers, and letting teams self-select? What governs the decision and how do you sequence the rollout?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32797_You_re_setting_the_org_s_messaging_strategy_for_the_next_thr.md>)

## Related concepts

- [Kafka Capacity and Cost](<../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/Kafka_Capacity_and_Cost.md>) *(Kafka Core)*
- [Kafka and the Lakehouse](<../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_and_the_Lakehouse.md>) *(Kafka Core)*
- [Consuming Untrusted Streams](<../Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*

## References

- *System Design Big Archive (2023)* — ByteByteGo — Why is Kafka fast

