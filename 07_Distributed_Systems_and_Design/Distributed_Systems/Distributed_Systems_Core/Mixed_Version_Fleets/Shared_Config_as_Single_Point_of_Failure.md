---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Shared_Config_as_Single_Point_of_Failure"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Redis"]
---

# Shared Config as Single Point of Failure

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Your agent's tool registry is a distributed config service read on every step, and a config-store blip spiked agent latency fleet-wide. Explain how you'd cache the registry locally with a TTL and safe fallback so the store is never on the critical path.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23157_Your_agent_s_tool_registry_is_a_distributed_config_service_r.md>)
- [Intermediate] [You manage a fleet where deploys are gated by a global feature-flag service that every pod reads. Explain how you'd keep flag reads from becoming a correlated SPOF and how a bad flag flip should be blast-radius-scoped.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23445_You_manage_a_fleet_where_deploys_are_gated_by_a_global_featu.md>)
- [Intermediate] [Your detectors read config (thresholds, model routing) from a config store on the hot path and a store blip spiked scoring latency fleet-wide. Explain how you'd cache config locally with a TTL and safe fallback so the config store is never in the critical detection path.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23916_Your_detectors_read_config_thresholds_model_routing_from_a_c.md>)
- [Deep] [Explain how a single bad config or model-version flip could propagate instantly to your entire inference fleet and cause a global incident. How would you design staged rollout and blast-radius scoping for model and config changes?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23705_Explain_how_a_single_bad_config_or_model-version_flip_could_.md>)
- [Deep] [Explain the operational danger of watch-based config or discovery systems where a reconnecting client re-lists the entire keyspace on a control-plane blip, and how that self-inflicted load turns a small hiccup into a coordinated stampede. How do you design the watch/resync protocol to be safe?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23516_Explain_the_operational_danger_of_watch-based_config_or_disc.md>)
- [Expert] [A postmortem shows a cascading failure that hopped from the caching tier to the database to the auth service and finally to DNS. Walk me through how you'd build the causal chain, identify the amplifiers at each hop, and turn the retro into a small number of structural fixes rather than 30 action items nobody does.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23330_A_postmortem_shows_a_cascading_failure_that_hopped_from_the_.md>)
- [Expert] [A single bad feature-flag flip caused a global incident because it propagated instantly to every service. Design a flag-delivery system that combines fast propagation with blast-radius control: staged rollout of flag changes, automatic rollback on SLO regression, and per-cell flag scoping.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23396_A_single_bad_feature-flag_flip_caused_a_global_incident_beca.md>)
- [Expert] [Explain how a single bad model-version or routing-config flip could propagate instantly to your entire inference fleet and cause a global incident. Design a staged, blast-radius-scoped rollout for model and config changes with automatic SLO-and-quality-based rollback, and why the flag system itself must not be a hot-path SPOF.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23785_Explain_how_a_single_bad_model-version_or_routing-config_fli.md>)

## Related concepts

- [Backpressure and Load Shedding](<../../../System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Standard Deployment Strategies](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Standard_Deployment_Strategies.md>) *(CI CD)*
- [Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>) *(Cloud Fundamentals)*

## References

- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Cautions and limits
- *Data Engineering with Medallion Architecture* — Miki Eto — Cloud Migration and Coexistence Strategies › Related topics
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › Run without dependencies
- *Ultimate AWS Data Engineering* — Rathish Mohan — Building Fault-Tolerant Architectures › Introduction
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Management › Maintaining Configuration
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Backup and Recovery › Hardware failures
- *Designing Data-Intensive Applications* — Martin Kleppmann — B › C
- *Database Internals* — Alex Petrov — F

