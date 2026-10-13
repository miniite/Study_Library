---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Gating_Trade-offs"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Git_and_GitHub"]
---

# Deployment Gating Trade-offs

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)

## Practice questions

- [Basic] [Why can long-lived feature branches make integration harder and riskier?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07375_Why_can_long-lived_feature_branches_make_integration_harder_.md>)
- [Deep] [How would you fail a build when a new dependency introduces a disallowed software license?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06816_How_would_you_fail_a_build_when_a_new_dependency_introduces_.md>)
- [Deep] [A hiring-manager scenario: you're brought in to fix a team that deploys once a quarter and every release is an all-hands war room. Diagnose the likely root causes and lay out the first 90 days to change the delivery culture and mechanics.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07083_A_hiring-manager_scenario_you_re_brought_in_to_fix_a_team_th.md>)
- [Deep] [How would you reshape team topologies so delivery flow matches the target architecture when Conway's law has team boundaries cutting straight across the end-to-end deploy path, and what stream-aligned versus platform-team split would you propose to make the org ship the system you actually want?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07550_How_would_you_reshape_team_topologies_so_delivery_flow_match.md>)
- [Expert] [A principal proposes trunk-based development org-wide; another insists their regulated team needs release branches for audit. As the deciding architect, how do you reconcile continuous delivery with regulatory change-control?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07057_A_principal_proposes_trunk-based_development_org-wide_anothe.md>)
- [Expert] [Design a compatibility-matrix gate for a system-of-systems where any two services deployed at incompatible version pairs cause outages; how do you compute allowed version combinations, enforce them at deploy time, and keep the matrix from becoming a deploy-blocking bottleneck?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07113_Design_a_compatibility-matrix_gate_for_a_system-of-systems_w.md>)
- [Expert] [A principal disagreement: one architect wants every deploy to require an automated integration test against real downstream services; you argue that couples every team's velocity to every other's uptime. How do you frame and resolve the trade-off?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07189_A_principal_disagreement_one_architect_wants_every_deploy_to.md>)
- [Expert] [How would you architect an org-level flakiness-management program (detection, quarantine, ownership SLAs, budget) and defend it as a platform investment rather than each team's private problem?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07243_How_would_you_architect_an_org-level_flakiness-management_pr.md>)

## Related concepts

- [Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Backpressure and Load Shedding](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>) *(Low Level Design)*

## References

- *97 Things Every Programmer Should Know* — Kevlin Henney (ed.) — One Binary

