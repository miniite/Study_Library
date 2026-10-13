---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Cost_of_Exactly_Once"
level: 5
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics"
tools: ["Apache_Kafka"]
---

# Cost of Exactly Once

**L5 Expert** · [↑ Delivery Semantics](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [An architect on another team wants to build in-house exactly-once middleware. Run the build-vs-buy analysis against Kafka transactions, Pulsar, and cloud-native queues, weighting long-term maintenance and hiring risk.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32406_An_architect_on_another_team_wants_to_build_in-house_exactly.md>)
- [Deep] [Explain how consumer offset commit strategy (auto vs manual, before vs after processing) determines whether you get at-most-once or at-least-once, and design the org guidance that makes at-most-once impossible-by-default.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32450_Explain_how_consumer_offset_commit_strategy_auto_vs_manual_b.md>)
- [Deep] [Mentor a senior engineer who insists exactly-once is always the safest default. Teach them the mental model for reasoning about delivery semantics per use case, and give them a decision tree they can apply without you.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32488_Mentor_a_senior_engineer_who_insists_exactly-once_is_always_.md>)
- [Expert] [Quantify the real cost of exactly-once at scale: latency, throughput ceiling, storage, and operational complexity. When would you consciously choose at-least-once plus idempotent consumers instead, and how do you defend that to auditors?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32393_Quantify_the_real_cost_of_exactly-once_at_scale_latency_thro.md>)
- [Expert] [Justify to a FinOps-driven CFO why doubling your Kafka broker footprint to enable exactly-once is worth it for the ledger pipeline but not for the clickstream pipeline. Frame the argument in dollars-per-error-class.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32405_Justify_to_a_FinOps-driven_CFO_why_doubling_your_Kafka_broke.md>)
- [Expert] [Design a reconciliation-first architecture where you deliberately accept at-least-once and duplicates, and rely on periodic reconciliation to achieve eventual correctness. When is this preferable to inline exactly-once, and what are its failure modes?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32424_Design_a_reconciliation-first_architecture_where_you_deliber.md>)
- [Expert] [An architecture review challenges your claim of exactly-once. Design the chaos/verification test suite (dupe injection, kill-mid-commit, partition heal) that would actually prove or falsify the guarantee before you ship it.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32433_An_architecture_review_challenges_your_claim_of_exactly-once.md>)
- [Expert] [You're advising leadership on whether to adopt a vendor's 'exactly-once' managed service. Design the due-diligence questions and failure-injection tests that separate genuine guarantees from marketing, before signing a multi-year contract.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Q-32456_You_re_advising_leadership_on_whether_to_adopt_a_vendor_s_ex.md>)

## Related concepts

- [Idempotent Consumers](<../Event_Driven_Architecture/Fat_vs_Thin_Events/Idempotent_Consumers.md>) *(Event Driven Architecture)*
- [Optimistic Concurrency and Clocks](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*
- [Idempotent Order APIs](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*

