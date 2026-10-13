---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Git_and_GitHub", "Prometheus"]
---

# Deployment Observability

**L4 Advanced** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Push vs Pull Deployments](<Push_vs_Pull_Deployments.md>)
- [DORA Metrics](<DORA_Metrics.md>)

## Implementations (tools)

- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Prometheus](<../../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Basic] [What does it mean to store your Prometheus alert rules in git and deploy them through a pipeline instead of clicking in a UI?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06430_What_does_it_mean_to_store_your_Prometheus_alert_rules_in_gi.md>)
- [Basic] [What is the difference between deploying a change automatically and requiring a manual approval, for a change that affects alerting?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06479_What_is_the_difference_between_deploying_a_change_automatica.md>)
- [Intermediate] [Why might you tag a deployed monitoring config with the git commit hash, and how does that help during an incident?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06500_Why_might_you_tag_a_deployed_monitoring_config_with_the_git_.md>)
- [Intermediate] [Your alerting pipeline deploys through CI but the on-call escalation config lives in a SaaS UI; how would you bring that config under version control and pipeline review?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06588_Your_alerting_pipeline_deploys_through_CI_but_the_on-call_es.md>)
- [Deep] [What happens to in-flight requests during a deployment and how do you avoid dropping them?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07493_What_happens_to_in-flight_requests_during_a_deployment_and_h.md>)
- [Deep] [Your alert-correlation service ships new grouping rules through CI; how do you gate the deploy so a rule change that would collapse or explode alert groups is caught before it reaches on-call?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06541_Your_alert-correlation_service_ships_new_grouping_rules_thro.md>)
- [Deep] [Your team is moving from manual weekly alerting-rule changes to continuous deployment of rules; what pipeline capabilities (linting, replay tests, canary, auto-rollback) would you build first and why?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06646_Your_team_is_moving_from_manual_weekly_alerting-rule_changes.md>)
- [Expert] [How do you wire a pipeline to watch metrics for a few minutes post-deploy and auto-abort if they degrade?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06794_How_do_you_wire_a_pipeline_to_watch_metrics_for_a_few_minute.md>)

## Related concepts

- [Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>) *(GitHub Actions)*
- [Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Sync Notifications](<../../GitOps/GitOps_Fundamentals/Sync_Notifications.md>) *(GitOps)*

