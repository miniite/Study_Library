---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Leader_Election"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Kubernetes"]
---

# Leader Election

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [You run a distributed cron via Kubernetes CronJobs and observed missed and doubled runs during control-plane load. Explain the concurrencyPolicy and startingDeadlineSeconds semantics and how they cause each failure.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23441_You_run_a_distributed_cron_via_Kubernetes_CronJobs_and_obser.md>)
- [Deep] [Walk me through Raft's leader election step by step. Why the randomized election timeout, and what would go wrong if every follower used the same fixed timeout?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23220_Walk_me_through_Raft_s_leader_election_step_by_step_Why_the_.md>)
- [Deep] [You need to run a StatefulSet database with three replicas on Kubernetes. Explain how you'd guarantee stable network identity ordered startup and that a re-scheduled pod reattaches to its own PVC and not another replica's data.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23415_You_need_to_run_a_StatefulSet_database_with_three_replicas_o.md>)
- [Deep] [A Kubernetes rolling update of a leader-elected controller briefly ran two active leaders and double-processed jobs. Explain how the lease-based leader election in client-go can allow this during a terminationGracePeriod and how you'd fix it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23416_A_Kubernetes_rolling_update_of_a_leader-elected_controller_b.md>)
- [Deep] [You run Vault in HA mode with integrated Raft storage. Explain how unseal auto-unseal and leader step-down work during a node replacement and how you avoid a window where no node can serve secrets.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23423_You_run_Vault_in_HA_mode_with_integrated_Raft_storage_Explai.md>)
- [Deep] [Explain how you'd operate a distributed job scheduler that runs each job exactly once at its scheduled time across a fleet, tolerating node failures without duplicates or misses. What leader election and sharding do you use?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23488_Explain_how_you_d_operate_a_distributed_job_scheduler_that_r.md>)
- [Deep] [You run remediation as a leader-elected singleton via a Kubernetes lease, but during a stop-the-world GC pause a second worker acquired the lease and both ran the same runbook. Explain the safety gap in lease-based election and how a fencing token prevents the double-run.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23864_You_run_remediation_as_a_leader-elected_singleton_via_a_Kube.md>)
- [Deep] [Your incident-automation must run a given remediation exactly once at its scheduled maintenance window across a fleet, tolerating node failure, without duplicate or missed runs. Explain the leader-election and lease design and the concurrencyPolicy and startingDeadline semantics if you build it on CronJobs.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23910_Your_incident-automation_must_run_a_given_remediation_exactl.md>)

## Related concepts

- [Exactly Once Scheduled Jobs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Exactly_Once_Scheduled_Jobs.md>) *(Cloud Fundamentals)*
- [Two-level Cache Design](<../../../System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*
- [HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>) *(High Availability)*

## References

- *Think Distributed Systems* — Dominik Tornow — Distributed consensus › 10.6.3 Leader Election protocol
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Note
- *Database Internals* — Alex Petrov — Leader Election › Chapter 10. Leader Election
- *Database Internals* — Alex Petrov — Leader Election › Next-In-Line Failover
- *Database Internals* — Alex Petrov — Leader Election › Candidate/Ordinary Optimization
- *Database Internals* — Alex Petrov — Raft › Leader Role in Raft
- *Designing Data-Intensive Applications* — Martin Kleppmann — Membership and Coordination Services › Allocating work to nodes

