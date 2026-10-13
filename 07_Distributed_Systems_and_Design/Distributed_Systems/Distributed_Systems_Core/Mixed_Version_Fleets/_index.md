---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Mixed Version Fleets

**L4 Advanced** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Quorums and Consensus](<Quorums_and_Consensus.md>)
- [Verifying Live Migrations](<Verifying_Live_Migrations.md>)
- [Distributed Training Clusters](<Distributed_Training_Clusters.md>)
- [Workload Isolation on Shared Clusters](<Workload_Isolation_on_Shared_Clusters.md>)
- [Cross-AZ Traffic Costs](<Cross-AZ_Traffic_Costs.md>)
- [Event Schema Evolution](<Event_Schema_Evolution.md>)
- [Shared Config as Single Point of Failure](<Shared_Config_as_Single_Point_of_Failure.md>)
- [Sync vs Async Replication](<Sync_vs_Async_Replication.md>)
- [Multi-tenant vs Dedicated Clusters](<Multi-tenant_vs_Dedicated_Clusters.md>)
- [Static Stability](<Static_Stability.md>)
- [Federated vs Centralized Data Ownership](<Federated_vs_Centralized_Data_Ownership.md>)
- [Choosing an Event Backbone](<Choosing_an_Event_Backbone.md>)

## Practice questions

- [Intermediate] [Your A/B test for two prompts across a distributed fleet must keep a given user on the same variant even as replicas scale. Explain hash-bucketing for sticky assignment and why non-sticky assignment silently invalidates your experiment stats.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23161_Your_A_B_test_for_two_prompts_across_a_distributed_fleet_mus.md>)
- [Intermediate] [You promote a new guardrail model but half the fleet picks up the new version and half stays stale for minutes, so the same user gets inconsistent safety behavior. Explain the mixed-version risk and how you'd make the switch atomic from a user's view.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23834_You_promote_a_new_guardrail_model_but_half_the_fleet_picks_u.md>)
- [Deep] [You mirror production traffic to a candidate embedding model to compare retrieval quality without user impact. Explain how you'd diff the retrieved sets at scale, the sampling strategy that keeps it affordable, and how you avoid the shadow doubling your embedding cost.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23204_You_mirror_production_traffic_to_a_candidate_embedding_model.md>)
- [Deep] [Explain shadow deployment for an ML model. How do you mirror production traffic to the new model, compare its predictions against the live one at scale, and avoid the shadow inference doubling your GPU cost or corrupting downstream state?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23636_Explain_shadow_deployment_for_an_ML_model_How_do_you_mirror_.md>)
- [Deep] [You need to retrain a model on fresh data every hour and the training-serving handoff must never leave the fleet on a half-updated version. Explain the atomic-swap deployment and how you avoid a window where some replicas serve v1 and others v2 inconsistently.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23657_You_need_to_retrain_a_model_on_fresh_data_every_hour_and_the.md>)
- [Deep] [You promote a model, but half the fleet picks up the new version and half is stale for several minutes because of caching and staggered rollout. Explain the mixed-version risk (inconsistent predictions to the same user) and how you'd make the switch atomic.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23708_You_promote_a_model_but_half_the_fleet_picks_up_the_new_vers.md>)
- [Deep] [Explain how you'd architect a global model registry and artifact-distribution system so that promoting a model to production is a linearizable operation, but the 30GB weights propagate to hundreds of serving pods across regions without a thundering herd or a mixed-version window.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23725_Explain_how_you_d_architect_a_global_model_registry_and_arti.md>)
- [Deep] [You're running an A/B test across a distributed serving fleet and must guarantee a given user deterministically and stickily gets the same model variant even as replicas scale and rollouts happen. Explain the hash-bucketing design and why non-sticky assignment silently invalidates your experiment statistics.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23760_You_re_running_an_A_B_test_across_a_distributed_serving_flee.md>)

## Related concepts

- [Multi-cloud Strategy](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*
- [ML Model Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>) *(CI CD)*
- [Two-level Cache Design](<../../../System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

## References

- *Data Engineering with Medallion Architecture* — Miki Eto — MLOps for AI Model Deployment and Monitoring › Related topics
- *Data Engineering with Medallion Architecture* — Miki Eto — DevOps and CICD for Data Engineering › Concrete example
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Microsoft Variation and Prioritization of Experiments › Chapter 6. Microsoft Variation and Prioritization of Experiments
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Microsoft Variation and Prioritization of Experiments › Combining Variation and Prioritization
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Humanistic Chaos › Vary Real-World Events
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — The Experiment Selection Problem (and a Solution) › Choosing Experiments
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Types of Consistency Models
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Trade-offs in Consistency Models
- *The Enterprise Big Data Lake* — Alex Gorelik — Introduction to Big Data and Data Science › Change Management
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — SBP 8 -- automate cloud code snippet deployments with standard deployment scripted wrappers › The deployment model choices
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Machine learning (ML)artificial intelligence (AI) overview › Machine learning pipeline
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Model governance and compliance › Why are models rarely deployed?
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Other important challenges › Artificial intelligence
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Fit the model on the training data › Model monitoring and management

