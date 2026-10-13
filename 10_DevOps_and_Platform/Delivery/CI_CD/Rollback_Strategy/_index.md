---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Git_and_GitHub", "Prometheus"]
---

# Rollback Strategy

**L4 Advanced** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Canary and Progressive Delivery](<Canary_and_Progressive_Delivery.md>)
- [Standard Deployment Strategies](<Standard_Deployment_Strategies.md>)

## Implementations (tools)

- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Prometheus](<../../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Basic] [If a bad alert rule got deployed and started paging everyone, what is the simplest way to undo it using version control?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06436_If_a_bad_alert_rule_got_deployed_and_started_paging_everyone.md>)
- [Basic] [What does it mean to 'version' your alerting configuration, and why does versioning help when something breaks?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06469_What_does_it_mean_to_version_your_alerting_configuration_and.md>)
- [Intermediate] [Why do teams avoid letting people edit alert rules directly in the UI when the rules are supposed to come from git?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06502_Why_do_teams_avoid_letting_people_edit_alert_rules_directly_.md>)
- [Intermediate] [How would you deploy a change to your metrics retention/rollup so that detectors relying on long baselines aren't starved of history after the change?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06627_How_would_you_deploy_a_change_to_your_metrics_retention_roll.md>)
- [Deep] [A change to your agent's memory/state format breaks in-flight sessions on deploy; how would you sequence the rollout so existing conversations don't corrupt during the swap?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06411_A_change_to_your_agent_s_memory_state_format_breaks_in-fligh.md>)
- [Deep] [How would you design the pipeline so that a rollback is a first-class, one-command operation rather than 'redeploy the previous commit', including how you handle config and migration state?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06950_How_would_you_design_the_pipeline_so_that_a_rollback_is_a_fi.md>)
- [Expert] [How would you wire your CD pipeline to auto-rollback an LLM deploy when live faithfulness or user-thumbs-down rate spikes, and how do you avoid rolling back on normal metric noise?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06348_How_would_you_wire_your_CD_pipeline_to_auto-rollback_an_LLM_.md>)
- [Expert] [How would you design the pipeline so that a rollback also reverts the observability/alerting config that shipped with the change, so on-call isn't blinded by mismatched dashboards after a rollback?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07007_How_would_you_design_the_pipeline_so_that_a_rollback_also_re.md>)

## Related concepts

- [Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>) *(Git Core)*
- [Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>) *(Cloud Fundamentals)*
- [Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*

