---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/API_Composition_and_BFF"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: []
---

# API Composition and BFF

**L5 Expert** · [↑ System Design Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [Walk me through how you would break a monolith into two services as it grows.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42389_Walk_me_through_how_you_would_break_a_monolith_into_two_serv.md>)
- [Deep] [Two senior teams are in a stalemate: one insists on gRPC internal APIs, the other on REST/JSON for developer ergonomics. As the deciding architect, how do you adjudicate this without becoming the bottleneck for every future protocol choice?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42580_Two_senior_teams_are_in_a_stalemate_one_insists_on_gRPC_inte.md>)
- [Deep] [Design a chargeback/showback system so each team sees the true cost of the infrastructure they consume. How do you attribute shared costs (a common database, the mesh, logging) fairly, and how does this change team behavior?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42643_Design_a_chargeback_showback_system_so_each_team_sees_the_tr.md>)
- [Deep] [You must decide whether real-time features (presence, live counts, notifications) share one WebSocket infrastructure or each team runs its own. Argue the platform-vs-autonomy trade-off, and design the multiplexed connection layer if you centralize it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42713_You_must_decide_whether_real-time_features_presence_live_cou.md>)
- [Expert] [Design a schema-evolution and data-contract governance model between producers and 20 consumer teams so that a producer can ship a change on Tuesday without a war-room. Who owns the contract and how is it enforced in CI?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42592_Design_a_schema-evolution_and_data-contract_governance_model.md>)
- [Expert] [You must standardize on one message broker for the org but teams need queues, streams, and pub-sub. Compare Kafka, Pulsar, SQS/SNS, and NATS as a strategic platform choice, and defend how you'd handle the workloads the winner serves badly.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42606_You_must_standardize_on_one_message_broker_for_the_org_but_t.md>)
- [Expert] [Design the sequencing for retiring a shared 'god database' that 25 services read and write directly. How do you introduce service ownership of data, break the direct table access, and do it without a coordinated big-bang?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42633_Design_the_sequencing_for_retiring_a_shared_god_database_tha.md>)
- [Expert] [Design a strategy for an org drowning in microservices (300+, many barely maintained) to consolidate back toward well-sized services. How do you decide what to merge, what to kill, and how do you sell 'fewer services' as progress?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42671_Design_a_strategy_for_an_org_drowning_in_microservices_300_m.md>)

## Related concepts

- [Service Mesh Adoption](<../../High_Level_Design/Service_Mesh_Adoption.md>) *(High Level Design)*
- [Federated vs Centralized Data Ownership](<../../../Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Federated_vs_Centralized_Data_Ownership.md>) *(Distributed Systems Core)*
- [Cardinality Isolation](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Cardinality_Isolation.md>) *(Streaming Architecture)*

## References

- *Claude Code: The Definitive Guide to Agentic Development* — Multi-Agent Orchestration

