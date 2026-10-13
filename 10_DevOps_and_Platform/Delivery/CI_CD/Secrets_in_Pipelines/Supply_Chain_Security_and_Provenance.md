---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Supply_Chain_Security_and_Provenance"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: []
---

# Supply Chain Security and Provenance

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [Your self-hosted runners hold cloud credentials and a compromised dependency in one job could exfiltrate them; how would you scope per-job permissions and isolate runners so one bad job's blast radius is minimal?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07428_Your_self-hosted_runners_hold_cloud_credentials_and_a_compro.md>)
- [Expert] [How does provenance/attestation (like SLSA) strengthen a build pipeline?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06778_How_does_provenance_attestation_like_SLSA_strengthen_a_build.md>)
- [Expert] [You're asked to add SLSA-style provenance and artifact signing; what threats does each address, and how do you verify signatures at deploy time without slowing the pipeline?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06858_You_re_asked_to_add_SLSA-style_provenance_and_artifact_signi.md>)
- [Expert] [A self-hosted runner was compromised and could read other repos' secrets; what isolation and hardening would you put in place so one bad job can't pivot across the fleet?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06909_A_self-hosted_runner_was_compromised_and_could_read_other_re.md>)
- [Expert] [Your security org mandates that every artifact reaching prod carry SLSA level-3 provenance; you have 300 pipelines and a two-quarter deadline. Lay out the rollout, the enforcement point, and how you handle teams who can't comply in time.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07067_Your_security_org_mandates_that_every_artifact_reaching_prod.md>)
- [Expert] [You're responsible for the security posture of self-hosted runners that hold cloud credentials for the whole company; design the isolation, credential-scoping, and blast-radius-containment architecture so one compromised job can't pivot to prod or to another team's secrets.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07094_You_re_responsible_for_the_security_posture_of_self-hosted_r.md>)
- [Expert] [You inherit five CI systems from acquisitions with incompatible security models; design the multi-year consolidation so you converge security posture first (secrets, provenance, runner isolation) before converging tooling, and defend that ordering.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07218_You_inherit_five_CI_systems_from_acquisitions_with_incompati.md>)
- [Expert] [A principal probe on your SLSA-level-3 mandate across 300 pipelines with a two-quarter deadline: where do you put the enforcement point, and how do you handle teams who genuinely can't comply in time without either blocking them or gutting the guarantee?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07266_A_principal_probe_on_your_SLSA-level-3_mandate_across_300_pi.md>)

## Related concepts

- [Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Shared Secret Rotation](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Shared_Secret_Rotation.md>) *(Secrets Management)*

