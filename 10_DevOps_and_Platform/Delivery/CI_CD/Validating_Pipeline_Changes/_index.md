---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Prometheus", "Git_and_GitHub"]
---

# Validating Pipeline Changes

**L4 Advanced** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Test Data Governance](<Test_Data_Governance.md>)
- [Deployment Verification](<Deployment_Verification.md>)

## Implementations (tools)

- [Prometheus](<../../../../TOOLS/Prometheus/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)

## Practice questions

- [Basic] [What is a 'checkout' step at the start of a pipeline, and why does a monitoring-deploy pipeline need it?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06503_What_is_a_checkout_step_at_the_start_of_a_pipeline_and_why_d.md>)
- [Intermediate] [How would you run a pipeline job that checks a set of recording rules can be loaded, without actually pushing them to production?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06459_How_would_you_run_a_pipeline_job_that_checks_a_set_of_record.md>)
- [Intermediate] [What is a smoke test and where does it run in a deployment pipeline?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07581_What_is_a_smoke_test_and_where_does_it_run_in_a_deployment_p.md>)
- [Intermediate] [How would you enforce that every service exposes a standard /health and /version endpoint via a CI check, so the deploy tooling can universally verify rollouts?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06995_How_would_you_enforce_that_every_service_exposes_a_standard_.md>)
- [Deep] [A serving change swapped from a managed API to self-hosted vLLM; how would you make CI verify functional and cost parity before cutting production traffic over?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06367_A_serving_change_swapped_from_a_managed_API_to_self-hosted_v.md>)
- [Deep] [How would you deploy a Kafka Connect connector configuration change (e.g., a Debezium CDC connector) through CI/CD so a bad config can't silently stop replicating a table?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06688_How_would_you_deploy_a_Kafka_Connect_connector_configuration.md>)
- [Deep] [How would you fail a build when a new transitive dependency introduces a disallowed license, and how do you avoid false positives blocking developers on every dependency bump?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06887_How_would_you_fail_a_build_when_a_new_transitive_dependency_.md>)
- [Deep] [A deploy that scales a service down (fewer replicas) as part of a cost change caused a queue backlog you didn't notice until hours later; how would you make the pipeline validate capacity changes against real throughput?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07010_A_deploy_that_scales_a_service_down_fewer_replicas_as_part_o.md>)

## Related concepts

- [Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>) *(GitHub Actions)*
- [Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*

