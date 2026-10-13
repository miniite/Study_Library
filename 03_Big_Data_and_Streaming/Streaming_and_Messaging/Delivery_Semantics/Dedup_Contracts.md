---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Dedup_Contracts"
level: 5
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics"
tools: ["Apache_Kafka"]
---

# Dedup Contracts

**L5 Expert** · [↑ Delivery Semantics](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [Design the semantic contract for a dead-letter-queue strategy across the org: when a message is DLQ'd after retries, how do you preserve at-least-once without creating silent data loss or unbounded reprocessing loops?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32401_Design_the_semantic_contract_for_a_dead-letter-queue_strateg.md>)
- [Deep] [Explain how Conway's law shows up in delivery-semantics failures: when producer and consumer teams disagree on the contract, duplicates and losses emerge at the seam. Design the team-topology and contract-testing approach to fix the human root cause.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32436_Explain_how_Conway_s_law_shows_up_in_delivery-semantics_fail.md>)
- [Deep] [You're the tie-breaker on whether to adopt Pulsar's built-in dedup + effectively-once vs Kafka transactions for a green-field platform. Run the decision framework across ops maturity, ecosystem, and semantics fidelity.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32483_You_re_the_tie-breaker_on_whether_to_adopt_Pulsar_s_built-in.md>)
- [Expert] [Explain how consumer group rebalancing under at-least-once can produce duplicate processing even with perfectly correct code, and design the fencing/generation-id strategy to make reprocessing safe.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32408_Explain_how_consumer_group_rebalancing_under_at-least-once_c.md>)
- [Expert] [Adjudicate a cross-team dispute: the fraud team wants exactly-once, the analytics team wants max throughput, both consume the same topic. Design a topology that satisfies both without duplicating the ingest infrastructure.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32420_Adjudicate_a_cross-team_dispute_the_fraud_team_wants_exactly.md>)
- [Expert] [A duplicate storm during a broker upgrade caused a 10x amplification because each retry triggered downstream retries. Analyze the retry-amplification failure mode and design backpressure and dedup that break the feedback loop.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32437_A_duplicate_storm_during_a_broker_upgrade_caused_a_10x_ampli.md>)
- [Expert] [Design delivery semantics for a webhook-delivery product (Stripe-style) where you promise at-least-once to customers. How do you communicate the duplicate contract, sign for replay-safety, and cap retry storms against a slow customer endpoint?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32448_Design_delivery_semantics_for_a_webhook-delivery_product_Str.md>)
- [Expert] [Design a canary/rollout strategy for changing a topic's delivery semantics (at-least-once to exactly-once) in production without a flag day, given hundreds of independently-deployed consumers with mixed readiness.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32466_Design_a_canary_rollout_strategy_for_changing_a_topic_s_deli.md>)

## Related concepts

- [Idempotent Consumers](<../Event_Driven_Architecture/Fat_vs_Thin_Events/Idempotent_Consumers.md>) *(Event Driven Architecture)*
- [Fan-out Trade-offs](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Idempotent Order APIs](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*

