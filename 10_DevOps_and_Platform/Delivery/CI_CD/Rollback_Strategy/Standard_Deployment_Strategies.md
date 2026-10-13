---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Standard_Deployment_Strategies"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: []
---

# Standard Deployment Strategies

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [What does 'zero-downtime deploy' mean and why do users care about it?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07339_What_does_zero-downtime_deploy_mean_and_why_do_users_care_ab.md>)
- [Intermediate] [A team wants deploys to only happen during business hours for safety, but that batches risk into fewer, bigger releases; make the argument for continuous deploys with strong guardrails instead, backed by data you'd collect.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06952_A_team_wants_deploys_to_only_happen_during_business_hours_fo.md>)
- [Deep] [You need deploys to stagger across AZs so a bad rollout can't hit all three failure domains at once; how would you make the CD system failure-domain-aware rather than relying on team discipline?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07449_You_need_deploys_to_stagger_across_AZs_so_a_bad_rollout_can_.md>)
- [Expert] [How would you coordinate deploying a backwards-incompatible API change across producer and consumer services?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06786_How_would_you_coordinate_deploying_a_backwards-incompatible_.md>)
- [Expert] [You're accountable for a delivery platform serving 10000 engineers across 15 regions; define its SLOs, its error budget, and what you'd do when the pipeline platform itself blows its budget during a critical release window.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07061_You_re_accountable_for_a_delivery_platform_serving_10000_eng.md>)
- [Expert] [Trade-off adjudication: for a global payments platform, argue where you'd sit on the spectrum between fast continuous deployment and heavily-gated staged rollout, and how you'd quantify the acceptable risk given regulatory and money-movement stakes.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07088_Trade-off_adjudication_for_a_global_payments_platform_argue_.md>)
- [Expert] [Incident retro at scale: a merge queue you introduced improved main stability but during a traffic-heavy launch it serialized deploys and delayed a critical fix by 40 minutes. How do you redesign the queue to preserve stability while allowing priority preemption for urgent changes?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07107_Incident_retro_at_scale_a_merge_queue_you_introduced_improve.md>)
- [Expert] [How do you architect deploy safety so that the blast radius of a bad config push (which needs no build) is bounded the same way a code deploy is, given config often bypasses the pipeline's staged rollout entirely?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07186_How_do_you_architect_deploy_safety_so_that_the_blast_radius_.md>)

## Related concepts

- [Backpressure and Load Shedding](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>) *(Cloud Fundamentals)*
- [Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>) *(Cloud Fundamentals)*

