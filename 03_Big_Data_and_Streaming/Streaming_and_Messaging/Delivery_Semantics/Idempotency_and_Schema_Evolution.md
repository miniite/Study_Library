---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Idempotency_and_Schema_Evolution"
level: 5
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics"
tools: ["Redis", "Apache_Kafka"]
---

# Idempotency and Schema Evolution

**L5 Expert** · [↑ Delivery Semantics](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [Compare broker-side dedup versus consumer-side idempotency as an organizational default. Which do you standardize on for a paved-road platform, and how do you keep app teams from silently violating the contract?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32399_Compare_broker-side_dedup_versus_consumer-side_idempotency_a.md>)
- [Deep] [An architect proposes using database-level unique constraints as the dedup mechanism instead of a dedup store. Adjudicate this: when is 'let the DB reject the duplicate insert' the right idempotency primitive, and when does it fail?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32460_An_architect_proposes_using_database-level_unique_constraint.md>)
- [Expert] [During an incident, a bug caused every downstream consumer to reprocess 12 hours of a topic, double-charging customers. Reconstruct the retro: what layered defenses (idempotency, dedup, reconciliation) should have contained the blast radius?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32392_During_an_incident_a_bug_caused_every_downstream_consumer_to.md>)
- [Expert] [A consumer's idempotency store is a single-region Redis, but the platform is going multi-region active-active. Redesign dedup so it survives region failover without either duplicating or dropping messages during split-brain.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32396_A_consumer_s_idempotency_store_is_a_single-region_Redis_but_.md>)
- [Expert] [Design a cross-system exactly-once flow: Kafka to a relational DB to an external payment gateway. Where do you place the idempotency boundary so the non-transactional external call never double-executes?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32409_Design_a_cross-system_exactly-once_flow_Kafka_to_a_relationa.md>)
- [Expert] [A skeptical staff engineer says CDC-based outbox is enough and you don't need application idempotency. Construct the scenario where CDC still delivers duplicates downstream and idempotency remains mandatory.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32414_A_skeptical_staff_engineer_says_CDC-based_outbox_is_enough_a.md>)
- [Expert] [Design the idempotency semantics for a mutation API behind the message bus where clients supply the key. How do you handle key reuse across different payloads, and what's your response contract for a retried-but-different request?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32422_Design_the_idempotency_semantics_for_a_mutation_API_behind_t.md>)
- [Expert] [An interviewer asks: 'Your idempotency store is now your bottleneck at peak.' Redesign for horizontal scale: sharding the dedup keyspace, avoiding hot keys, and keeping cross-shard correctness under rebalancing.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32442_An_interviewer_asks_Your_idempotency_store_is_now_your_bottl.md>)

## Related concepts

- [Idempotent Ingestion](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>) *(Distributed Systems Core)*
- [Idempotent Consumers](<../Event_Driven_Architecture/Fat_vs_Thin_Events/Idempotent_Consumers.md>) *(Event Driven Architecture)*
- [Idempotent Order APIs](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns

