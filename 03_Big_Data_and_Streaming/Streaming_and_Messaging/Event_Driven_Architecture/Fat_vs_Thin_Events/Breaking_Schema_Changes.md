---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Breaking_Schema_Changes"
level: 5
type: "deep-dive"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture"
tools: ["Apache_Kafka"]
---

# Breaking Schema Changes

**L5 Expert** · [↑ Event Driven Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [What are the pros and cons of event-driven architecture compared to a request-response monolith?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32592_What_are_the_pros_and_cons_of_event-driven_architecture_comp.md>)
- [Deep] [A new event schema field is added as required by one team and it silently breaks another team's consumer in production. How do you set up the org and tooling so this can't happen again?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32728_A_new_event_schema_field_is_added_as_required_by_one_team_an.md>)
- [Deep] [Standardization vs autonomy: some teams want to use SQS, some NATS, some Kafka, each with a real justification. How do you set a paved-road policy that permits deviation without fragmenting the org's operational and observability tooling?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32809_Standardization_vs_autonomy_some_teams_want_to_use_SQS_some_.md>)
- [Deep] [You need to influence three peer teams to adopt a shared event schema they didn't design and don't love. You have no authority over them. Walk me through your actual strategy to land this without a mandate, and what you'd concede to get buy-in.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32877_You_need_to_influence_three_peer_teams_to_adopt_a_shared_eve.md>)
- [Deep] [How would you architect an event platform so a single team can safely run a 'shadow' consumer against production event streams to prototype a feature, with zero risk of side effects, resource starvation, or offset interference on the real consumers?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32888_How_would_you_architect_an_event_platform_so_a_single_team_c.md>)
- [Expert] [Define the error-budget and SLO framework for a shared event platform used by dozens of teams. How do you attribute a lag breach to the platform versus a tenant, and what does the budget policy trigger for each?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32801_Define_the_error-budget_and_SLO_framework_for_a_shared_event.md>)
- [Expert] [How do you architect data governance and lineage across an event backbone so that when a regulator asks 'where did this customer's data flow', you can answer across every topic and consumer without instrumenting each team by hand?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32825_How_do_you_architect_data_governance_and_lineage_across_an_e.md>)
- [Expert] [Architect a real-time collaborative editing backend (think docs/whiteboard) on an event/log substrate where thousands edit the same document. How do ordering, conflict resolution, and per-document partitioning interact, and where does the messaging layer stop and CRDT/OT begin?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32854_Architect_a_real-time_collaborative_editing_backend_think_do.md>)

## Related concepts

- [API Composition and BFF](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>) *(System Design Core)*
- [Cardinality Isolation](<../../Streaming_Architecture/Provisional_and_Final_Results/Cardinality_Isolation.md>) *(Streaming Architecture)*
- [Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*

