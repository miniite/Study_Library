---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Zero_Downtime_Deployments"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Kubernetes"]
---

# Zero Downtime Deployments

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [Your detector deploy is fine but the model artifact is 4GB and pull time delays rollout during an incident; how would you optimize the artifact and pipeline so redeploys are fast when it matters?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06611_Your_detector_deploy_is_fine_but_the_model_artifact_is_4GB_a.md>)
- [Deep] [Your observability-as-code changes deploy to 30 clusters; how would you stagger the rollout across failure domains so a bad collector config can't blind your whole fleet at once?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06581_Your_observability-as-code_changes_deploy_to_30_clusters_how.md>)
- [Deep] [During a rolling model deploy, warm-up (weight load, JIT/graph compile) causes the new pod's first requests to be slow; how do you handle warm-up in CD?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07684_During_a_rolling_model_deploy_warm-up_weight_load_JIT_graph_.md>)
- [Deep] [Your rollout uses a Kubernetes Deployment with maxSurge and maxUnavailable defaults and it briefly halves capacity at peak traffic; how would you tune the rollout so it never dips below what live traffic needs?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07388_Your_rollout_uses_a_Kubernetes_Deployment_with_maxSurge_and_.md>)
- [Deep] [A rolling deploy of a service behind an NLB returns connection resets for a few seconds as old pods terminate; how would you use preStop hooks and deregistration delay to close that window?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07397_A_rolling_deploy_of_a_service_behind_an_NLB_returns_connecti.md>)
- [Deep] [A deploy's readiness probe was too eager and marked pods ready before caches warmed, spiking p99 for the first minute; how would you sequence cache warm-up before a pod takes traffic?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07429_A_deploy_s_readiness_probe_was_too_eager_and_marked_pods_rea.md>)
- [Expert] [How do you achieve graceful shutdown for a service handling long-lived connections (websockets/streams) during a rolling deploy without dropping users mid-session?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06901_How_do_you_achieve_graceful_shutdown_for_a_service_handling_.md>)
- [Expert] [A deploy that changes connection pool sizing exhausts the database's max connections when old and new pods overlap during rollout; how do you account for transient double-capacity in your deploy plan?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06991_A_deploy_that_changes_connection_pool_sizing_exhausts_the_da.md>)

## Related concepts

- [Org-wide Backpressure](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>) *(Cloud Fundamentals)*
- [Slow Dependencies and Load Shedding](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>) *(Distributed Systems Core)*

