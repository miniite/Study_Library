---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Canary_and_Progressive_Delivery"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: []
---

# Canary and Progressive Delivery

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [In simple terms, what is a canary deployment and what is the point of it?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07342_In_simple_terms_what_is_a_canary_deployment_and_what_is_the_.md>)
- [Deep] [How would you deploy a new LLM router (model-selection logic) so that a routing bug can only send a small fraction of traffic to the wrong/expensive model before you catch it?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06359_How_would_you_deploy_a_new_LLM_router_model-selection_logic_.md>)
- [Deep] [How do you decide between deploying a new anomaly detector as a canary gated on false-positive rate versus a shadow deploy gated by comparison against the incumbent?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06553_How_do_you_decide_between_deploying_a_new_anomaly_detector_a.md>)
- [Deep] [Your team wants to move from weekly big-bang releases to continuous deployment but leadership fears instability; lay out the concrete pipeline capabilities (tests, canary, flags, rollback, observability) you'd build first and the order you'd build them to earn that trust.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07046_Your_team_wants_to_move_from_weekly_big-bang_releases_to_con.md>)
- [Expert] [How would you auto-promote a canary model based on live business metrics (CTR, conversion) rather than just error rate, and what's the risk of gating on business metrics?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07624_How_would_you_auto-promote_a_canary_model_based_on_live_busi.md>)
- [Expert] [A canary looks healthy on error rate and latency but the new version leaks file descriptors and dies an hour after full rollout; how would you extend your canary or post-deploy checks to catch slow-burn resource leaks?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07389_A_canary_looks_healthy_on_error_rate_and_latency_but_the_new.md>)
- [Expert] [A skeptical staff engineer argues canary and auto-rollback are over-engineering for your traffic and rolling deploys are enough; as the architect you disagree for tier-1 services. Steelman their case, then make the decision and the criteria that determine per-service which model applies.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07150_A_skeptical_staff_engineer_argues_canary_and_auto-rollback_a.md>)
- [Expert] [How do you architect the delivery of a service mesh's own control-plane changes so that a bad mesh config (which affects routing for everyone) is canaried and blast-radius-bounded like any other deploy?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07250_How_do_you_architect_the_delivery_of_a_service_mesh_s_own_co.md>)

## Related concepts

- [Sensitivity Points](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>) *(Trade off Analysis)*
- [Backpressure and Load Shedding](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Utility_Trees.md>) *(Trade off Analysis)*
- [Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>) *(Cloud Fundamentals)*
- [Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>) *(Cloud Fundamentals)*

## References

- *System Design: The Big Archive* — Alex Xu (ByteByteGo) — Deployment strategies

