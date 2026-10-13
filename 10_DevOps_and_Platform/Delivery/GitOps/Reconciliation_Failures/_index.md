---
id: "10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/GitOps"
tools: ["Kubernetes", "Git_and_GitHub", "Argo_CD"]
---

# Reconciliation Failures

**L4 Advanced** · [↑ GitOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Resilience to Git Outages](<Resilience_to_Git_Outages.md>)

## Implementations (tools)

- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Argo CD](<../../../../TOOLS/Argo_CD/_index.md>)

## Practice questions

- [Intermediate] [What is configuration drift and how does GitOps detect it?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29091_What_is_configuration_drift_and_how_does_GitOps_detect_it.md>)
- [Intermediate] [What could go wrong if two people edit the cluster manually while GitOps is running?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29111_What_could_go_wrong_if_two_people_edit_the_cluster_manually_.md>)
- [Intermediate] [How would you debug a GitOps application that stays 'OutOfSync' after a commit?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29123_How_would_you_debug_a_GitOps_application_that_stays_OutOfSyn.md>)
- [Intermediate] [How would you know if a GitOps sync has been failing repeatedly?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29171_How_would_you_know_if_a_GitOps_sync_has_been_failing_repeate.md>)
- [Deep] [A Kustomize overlay in production silently stopped applying a patch after a base upgrade — how would you catch this class of drift before it hits prod?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29188_A_Kustomize_overlay_in_production_silently_stopped_applying_.md>)
- [Deep] [A failed PostSync hook Job left an Application stuck in Progressing forever — how do you debug hook lifecycle and clean up orphaned hook resources?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29195_A_failed_PostSync_hook_Job_left_an_Application_stuck_in_Prog.md>)
- [Deep] [How do you handle a config change that must be applied to 150 clusters, some of which are temporarily unreachable, while keeping git as truth?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29201_How_do_you_handle_a_config_change_that_must_be_applied_to_15.md>)
- [Deep] [A sync of a large CRD-heavy Helm release fails with 'the server could not find the requested resource' on first apply — why does ordering break here and how do you fix it?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29242_A_sync_of_a_large_CRD-heavy_Helm_release_fails_with_the_serv.md>)

## Related concepts

- [Push vs Pull Deployments](<../../CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>) *(CI CD)*
- [Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>) *(Git Core)*
- [Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>) *(Git Core)*

