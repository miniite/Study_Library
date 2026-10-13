---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Slow_Dependencies_and_Load_Shedding"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Slow Dependencies and Load Shedding

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Your agent stores conversation state in a distributed cache and a cache node failover loses a few seconds of recent turns, so the agent 'forgets' mid-conversation. Walk me through making conversation state durable without adding unacceptable per-turn latency.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23818_Your_agent_stores_conversation_state_in_a_distributed_cache_.md>)
- [Intermediate] [Your detector's readiness probe passes but it can't actually score because the model failed to load, so the LB routes telemetry to a zombie that silently drops detection. Explain deep versus shallow health checks here and the crash-loop risk of making liveness too deep.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23896_Your_detector_s_readiness_probe_passes_but_it_can_t_actually.md>)
- [Deep] [A single slow dependency (p99 of 2s) is causing your service's threads to all block, cascading into a full outage. Explain how bulkheads and timeouts isolate this, and how you'd right-size the timeout.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23263_A_single_slow_dependency_p99_of_2s_is_causing_your_service_s.md>)
- [Deep] [You need admission control on the hot path to shed non-critical work during overload. Explain how you'd keep the shedding decision cheap and avoid shedding the health-check and control-plane traffic you need to recover.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23479_You_need_admission_control_on_the_hot_path_to_shed_non-criti.md>)
- [Deep] [Explain why adding capacity to a metastably-overloaded system can make it worse, not better, and enumerate the counter-intuitive first interventions (shed load, cap concurrency, admission control) you'd reach for before scaling out during an active incident.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23552_Explain_why_adding_capacity_to_a_metastably-overloaded_syste.md>)
- [Expert] [Why is it impossible to reliably distinguish a slow node from a dead node in an asynchronous network? Explain how this fundamental limit shapes the design of failure detectors and timeouts.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23234_Why_is_it_impossible_to_reliably_distinguish_a_slow_node_fro.md>)
- [Expert] [Explain why 'we have backups' is not a DR strategy and 'we have a hot standby' is not automatically resilient. Walk me through the failure modes each hides (untested restores, correlated corruption replicated to the standby, failover that never fires) and how you'd validate real recoverability.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23378_Explain_why_we_have_backups_is_not_a_DR_strategy_and_we_have.md>)
- [Expert] [Explain how a virtual waiting room protects downstreams during a surge and the subtle ways it becomes the correlated SPOF itself. How do you size and shard it, and what's the plan if the waiting-room service is exactly what fails under the surge it was meant to absorb?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23607_Explain_how_a_virtual_waiting_room_protects_downstreams_duri.md>)

## Related concepts

- [Org-wide Backpressure](<../../../System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Graceful Degradation in Streaming](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>) *(Streaming Architecture)*
- [Fan-out Trade-offs](<../../../System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Fault tolerance and resilience
- *Think Distributed Systems* — Dominik Tornow — Message delivery and processing › 4.3 Silence and chatter
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Consequences for application design
- *Hadoop: The Definitive Guide* — Tom White — Flume › Note
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Queue jam
- *Database Internals* — Alex Petrov — Introduction and Overview › Cascading Failures
- *Database Internals* — Alex Petrov — Failure Detection › Chapter 9. Failure Detection
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Performance and latency

