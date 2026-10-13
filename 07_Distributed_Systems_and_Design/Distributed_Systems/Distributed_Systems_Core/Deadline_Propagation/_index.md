---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Deadline Propagation

**L4 Advanced** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Slow Dependencies and Load Shedding](<Slow_Dependencies_and_Load_Shedding.md>)

## Practice questions

- [Intermediate] [Your feature service for a ranking model reads from an online store at p99 of 20ms but the model needs p99 of 50ms end-to-end including generation. Walk me through how you'd budget the deadline across hops and what you'd cut or parallelize.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23150_Your_feature_service_for_a_ranking_model_reads_from_an_onlin.md>)
- [Intermediate] [Explain how you'd tune a client-side timeout for a downstream service given its p50 p99 and p999 latency. Why does a timeout set at p99 cause a retry storm during a latency shift?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23456_Explain_how_you_d_tune_a_client-side_timeout_for_a_downstrea.md>)
- [Deep] [You batch requests to an LLM endpoint for throughput, but a single very long generation in the batch holds up all the shorter ones. Explain this head-of-line blocking in static batching and how continuous batching mitigates it, and what scheduling problem the router now owns.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23821_You_batch_requests_to_an_LLM_endpoint_for_throughput_but_a_s.md>)
- [Deep] [Explain how you'd choose a client statement_timeout for a query against a distributed database given its p50/p99/p999 latency. Why does setting it at p99 trigger a retry storm the moment the latency distribution shifts?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24139_Explain_how_you_d_choose_a_client_statement_timeout_for_a_qu.md>)
- [Deep] [Explain how deadline propagation should work across a gRPC call chain in a microservice mesh and what wasted work and retry amplification you cause during an incident if each hop resets its own timeout.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23434_Explain_how_deadline_propagation_should_work_across_a_gRPC_c.md>)
- [Deep] [Explain deadline propagation for an inference request that fans out to a feature store, a ranking model, and a re-ranker. Why should each hop respect the remaining deadline rather than reset its own timeout, and what wasted GPU work results if it doesn't?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23662_Explain_deadline_propagation_for_an_inference_request_that_f.md>)
- [Deep] [You must right-size timeouts org-wide from a downstream's p50/p99/p999. Explain why a timeout set at p99 causes a retry storm the instant the latency distribution shifts, and how you'd set adaptive, load-aware timeouts as a platform default instead of a static number.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23557_You_must_right-size_timeouts_org-wide_from_a_downstream_s_p5.md>)
- [Deep] [Explain how you'd architect deadline propagation through your telemetry-processing DAG so a slow enrichment step doesn't cause the whole pipeline to buffer unboundedly and eventually collapse. Why must each stage respect a remaining budget rather than block indefinitely?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23962_Explain_how_you_d_architect_deadline_propagation_through_you.md>)

## Related concepts

- [Complexity of Retries](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>) *(Complexity Analysis)*
- [Fan-out Trade-offs](<../../../System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Lag-based Consumer Autoscaling](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>) *(Event Driven Architecture)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Latency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Latency and response time
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unreliable Networks › Timeouts and Unbounded Delays
- *Designing Data-Intensive Applications* — Martin Kleppmann — Synchronous Versus Asynchronous Networks › Can we not simply make network delays predictable?
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Why every millisecond matters
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Arbitrary Stateful Computations › Processing-time timeouts

