---
id: "10_DevOps_and_Platform/Delivery/GitOps/Scaling_Argo_CD"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/GitOps"
tools: ["Argo_CD", "Kubernetes", "Git_and_GitHub", "Prometheus"]
---

# Scaling Argo CD

**L5 Expert** · [↑ GitOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Argo CD](<../../../../TOOLS/Argo_CD/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Prometheus](<../../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Intermediate] [How would you check the logs of the Argo CD controller if a sync fails?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29124_How_would_you_check_the_logs_of_the_Argo_CD_controller_if_a_.md>)
- [Intermediate] [How would you detect and alert on 'apps that have been OutOfSync for more than 15 minutes' across a fleet — what metrics does Argo CD expose?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29245_How_would_you_detect_and_alert_on_apps_that_have_been_OutOfS.md>)
- [Deep] [A teammate ran kubectl edit on a production Deployment and Argo CD self-heal reverted it mid-incident, worsening the outage — how do you reconcile emergency changes with self-heal safely?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29184_A_teammate_ran_kubectl_edit_on_a_production_Deployment_and_A.md>)
- [Deep] [A sudden burst of 2000 commits (a bulk refactor) overwhelmed the Argo CD repo-server and syncs backed up — how do you diagnose repo-server resource pressure and scale it?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29221_A_sudden_burst_of_2000_commits_a_bulk_refactor_overwhelmed_t.md>)
- [Deep] [A dependency between two Argo CD Applications (app B needs app A's service) causes flapping health on deploy — how do you express cross-application ordering?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29251_A_dependency_between_two_Argo_CD_Applications_app_B_needs_ap.md>)
- [Deep] [Your org runs both Flux and Argo CD in different teams and wants to consolidate — how would you evaluate migration cost and risk of standardizing on one?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29275_Your_org_runs_both_Flux_and_Argo_CD_in_different_teams_and_w.md>)
- [Expert] [Mutating webhooks (Istio sidecar injection) cause Argo CD to see constant diffs on pod specs — how do you configure ignoreDifferences correctly without masking real drift?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29211_Mutating_webhooks_Istio_sidecar_injection_cause_Argo_CD_to_s.md>)
- [Expert] [Your Argo CD instance became a bottleneck and you're deciding between one big instance versus sharded instances per cluster — what drives that architecture choice?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29270_Your_Argo_CD_instance_became_a_bottleneck_and_you_re_decidin.md>)

## Related concepts

- [Push vs Pull Deployments](<../../CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>) *(CI CD)*
- [Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>) *(Git Core)*
- [Shared Config as Single Point of Failure](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Shared_Config_as_Single_Point_of_Failure.md>) *(Distributed Systems Core)*

