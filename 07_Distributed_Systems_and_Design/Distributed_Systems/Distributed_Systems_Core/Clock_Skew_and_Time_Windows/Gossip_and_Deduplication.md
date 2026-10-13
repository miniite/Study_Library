---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Gossip_and_Deduplication"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Gossip and Deduplication

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You correlate alerts by a topology graph fetched from a CMDB on the hot path, and a CMDB blip spiked correlation latency and split incidents. Explain how you'd cache the topology locally with a TTL and safe fallback so the CMDB is never in the critical correlation path.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23856_You_correlate_alerts_by_a_topology_graph_fetched_from_a_CMDB.md>)
- [Deep] [Your multi-node fine-tuning job silently hangs with no error using NCCL. Walk me through isolating a mismatched collective, a single crashed rank the others wait on, or a deadlocked all-reduce, and the timeouts and watchdogs you'd add to fail fast.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23179_Your_multi-node_fine-tuning_job_silently_hangs_with_no_error.md>)
- [Deep] [Explain why a heartbeat interval and a session timeout must be tuned together in ZooKeeper/etcd-backed leadership. What happens on a long GC pause, and how does that cause a spurious leader change?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23286_Explain_why_a_heartbeat_interval_and_a_session_timeout_must_.md>)
- [Deep] [Your alert-correlation engine groups related alerts into one incident, but a network partition split its two nodes and both opened a duplicate incident for the same outage. Explain how the partition caused the double-fire and how you'd design leadership so only one node correlates a given alert stream.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23849_Your_alert-correlation_engine_groups_related_alerts_into_one.md>)
- [Deep] [Your alert-correlation engine batches alerts into windows for grouping, and a single very long-running incident kept a window open, blocking correlation of newer unrelated alerts behind it. Explain this head-of-line blocking and how you'd bound window duration without splitting genuine incidents.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23900_Your_alert-correlation_engine_batches_alerts_into_windows_fo.md>)
- [Deep] [Your gossip-based agent-membership layer marked half the fleet dead during a cross-AZ latency blip, triggering a flood of false host-down alerts. Explain how suspicion timeouts and incarnation numbers dampen this and how you'd tune fanout and interval for a 10,000-agent fleet.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23942_Your_gossip-based_agent-membership_layer_marked_half_the_fle.md>)
- [Deep] [Explain the org-wide risk of synchronized clients: everyone refreshing tokens on the hour, every cron at midnight, all TTLs expiring together. Give the general thundering-herd pattern and the standards (deterministic jitter, staggered TTLs, request smoothing) you'd mandate to prevent it by construction.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23609_Explain_the_org-wide_risk_of_synchronized_clients_everyone_r.md>)
- [Deep] [You must right-size the alert-evaluation interval and detection window org-wide. Explain the tension between fast detection (short windows, more false positives from noise) and stable detection (long windows, slower to fire), and how you'd set adaptive, load-aware windows as a platform default.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24026_You_must_right-size_the_alert-evaluation_interval_and_detect.md>)

## Related concepts

- [Fan-out Trade-offs](<../../../System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Lag-based Consumer Autoscaling](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>) *(Event Driven Architecture)*
- [Duplicates During Rebalance](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Duplicates_During_Rebalance.md>) *(Streaming Architecture)*

## References

- *Hadoop: The Definitive Guide* — Tom White — Flume › Replicating and Multiplexing Selectors
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › The herd effect
- *Database Internals* — Alex Petrov — Failure Detection › Chapter 9. Failure Detection
- *Database Internals* — Alex Petrov — Failure Detection › Outsourced Heartbeats
- *Database Internals* — Alex Petrov — Failure Detection › Phi-Accural Failure Detector
- *Database Internals* — Alex Petrov — Failure Detection › Gossip and Failure Detection
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Leader failure: Failover
- *Designing Data-Intensive Applications* — Martin Kleppmann — Membership and Coordination Services
- *Ultimate AWS Data Engineering* — Rathish Mohan — Orchestrating Data Flow › SQS/SNS: The Messengers and Alert System
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Conclusion › D

