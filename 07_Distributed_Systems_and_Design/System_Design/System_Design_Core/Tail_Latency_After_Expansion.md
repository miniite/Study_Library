---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: ["Redis", "AWS"]
---

# Tail Latency After Expansion

**L4 Advanced** · [↑ System Design Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Your service's p99 is dominated by a slow N+1 in an ORM you can't easily rewrite. How do you find it in traces, batch it with a dataloader/IN query, and prevent regressions in review?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42505_Your_service_s_p99_is_dominated_by_a_slow_N_1_in_an_ORM_you_.md>)
- [Deep] [You added a cache but the app still feels slow — how would you investigate?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42280_You_added_a_cache_but_the_app_still_feels_slow_how_would_you.md>)
- [Deep] [Your JVM service shows periodic 400ms latency spikes correlated with full GC. How would you distinguish a memory leak from normal churn, and what heap/GC tuning would you try first?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42166_Your_JVM_service_shows_periodic_400ms_latency_spikes_correla.md>)
- [Deep] [Your Redis is single-threaded and one slow O(n) command (KEYS, big SMEMBERS) stalls everyone. How do you find the offending command, and what alternatives (SCAN, keyspace design) fix it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42497_Your_Redis_is_single-threaded_and_one_slow_O_n_command_KEYS_.md>)
- [Deep] [A microservice is causing a cascading slowdown but logs are inconclusive. Explain how distributed tracing (spans, trace context propagation) pinpoints it, and what you'd instrument to prove it.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42560_A_microservice_is_causing_a_cascading_slowdown_but_logs_are_.md>)
- [Deep] [Your cloud bill for a service doubled with no traffic change. Walk me through cost-attribution: cross-AZ data transfer, over-provisioned instances, chatty S3 calls, and idle read replicas — how do you find and cut the waste?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42575_Your_cloud_bill_for_a_service_doubled_with_no_traffic_change.md>)
- [Expert] [A read-heavy service is 95% cache hits but tail latency spiked after you added a new region. How would you localize whether the cause is cache node hot-keys, cross-region replication lag, or connection-pool exhaustion?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42394_A_read-heavy_service_is_95_cache_hits_but_tail_latency_spike.md>)
- [Expert] [Your p99 is fine but p99.9 is terrible and only for one endpoint. How do you methodically hunt the cause — GC pauses, lock contention, a slow dependency, or a single hot partition?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42419_Your_p99_is_fine_but_p99_9_is_terrible_and_only_for_one_endp.md>)

## Related concepts

- [Partition Lag and Hot Keys](<../../Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>) *(Distributed Systems Core)*
- [Tail Latency Investigation](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation.md>) *(Cloud Fundamentals)*
- [Complexity of Retries](<../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>) *(Complexity Analysis)*

