---
id: "10_DevOps_and_Platform/Delivery/GitOps/Multi-cluster_Ordering"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/GitOps"
tools: ["Kubernetes", "Git_and_GitHub", "Argo_CD", "AWS", "Terraform"]
---

# Multi-cluster Ordering

**L5 Expert** · [↑ GitOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Argo CD](<../../../../TOOLS/Argo_CD/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Terraform](<../../../../TOOLS/Terraform/_index.md>)

## Practice questions

- [Deep] [What is Crossplane and how does it enable GitOps for cloud resources?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29135_What_is_Crossplane_and_how_does_it_enable_GitOps_for_cloud_r.md>)
- [Deep] [How do you handle GitOps for resources that legitimately change at runtime, like HPA-scaled replica counts, without perpetual drift?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29226_How_do_you_handle_GitOps_for_resources_that_legitimately_cha.md>)
- [Deep] [As principal, how do you set an org-wide policy on what belongs in git versus what is legitimately runtime-owned state, and how do you enforce that boundary architecturally?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29284_As_principal_how_do_you_set_an_org-wide_policy_on_what_belon.md>)
- [Deep] [Design a fleet-wide compliance report answering 'what version is running where right now' with confidence, given reconcile lag makes git and live briefly diverge.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29337_Design_a_fleet-wide_compliance_report_answering_what_version.md>)
- [Deep] [How would you evaluate whether to adopt an OpenGitOps-conformant tool versus a vendor platform, and what conformance criteria actually matter for a 10-year platform bet?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29356_How_would_you_evaluate_whether_to_adopt_an_OpenGitOps-confor.md>)
- [Expert] [Your GitOps controller has cluster-admin on every cluster and security flagged it as a blast-radius risk — how would you scope its RBAC down without breaking reconciliation?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29202_Your_GitOps_controller_has_cluster-admin_on_every_cluster_an.md>)
- [Expert] [Design a fleet-wide reconciliation architecture for 2000 clusters across 6 regions — do you shard controllers per region, per tenant, or per cluster, and what drives the topology?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29285_Design_a_fleet-wide_reconciliation_architecture_for_2000_clu.md>)
- [Expert] [Reconcile GitOps with two competing control planes — Terraform/Crossplane for cloud infra and Argo CD for workloads — where each legitimately owns part of the state; how do you draw the seam?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29299_Reconcile_GitOps_with_two_competing_control_planes_Terraform.md>)

## Related concepts

- [Push vs Pull Deployments](<../../CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>) *(CI CD)*
- [Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>) *(Git Core)*
- [Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>) *(Git Core)*

