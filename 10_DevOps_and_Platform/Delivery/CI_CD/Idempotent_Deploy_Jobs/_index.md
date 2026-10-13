---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Git_and_GitHub", "AWS", "Jenkins"]
---

# Idempotent Deploy Jobs

**L4 Advanced** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Zero Downtime Deployments](<Zero_Downtime_Deployments.md>)
- [Expand and Contract Migrations](<Expand_and_Contract_Migrations.md>)
- [Database Migrations in Deployments](<Database_Migrations_in_Deployments.md>)

## Implementations (tools)

- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Jenkins](<../../../../TOOLS/Jenkins/_index.md>)

## Practice questions

- [Intermediate] [When a pipeline deploys a new version of a log-shipper, why is it important that the old logs keep flowing during the switch?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06454_When_a_pipeline_deploys_a_new_version_of_a_log-shipper_why_i.md>)
- [Intermediate] [Your integration tests hit a shared staging database and two concurrent pipeline runs corrupt each other's fixtures; how do you give each run isolation without provisioning a full database per run?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06942_Your_integration_tests_hit_a_shared_staging_database_and_two.md>)
- [Deep] [Your deploy job succeeds but half the pods run the old image. Where do you look?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06802_Your_deploy_job_succeeds_but_half_the_pods_run_the_old_image.md>)
- [Deep] [Your Airflow deploy replaced a DAG whose schedule_interval or start_date changed and it triggered a flood of backfill runs; how would you guard schedule changes in CI to prevent accidental mass backfills?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06659_Your_Airflow_deploy_replaced_a_DAG_whose_schedule_interval_o.md>)
- [Deep] [How would you make your data pipeline deploy idempotent so that re-running the CD job (after a transient failure) doesn't double-insert rows or re-trigger a backfill?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06724_How_would_you_make_your_data_pipeline_deploy_idempotent_so_t.md>)
- [Deep] [A deploy job holds a distributed lock so only one rollout of a service happens at a time, but a crashed job left the lock held and blocked all deploys; how do you make deploy locks self-healing?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06944_A_deploy_job_holds_a_distributed_lock_so_only_one_rollout_of.md>)
- [Deep] [You use Terraform in the deploy pipeline and a plan shows a spurious replace of an RDS instance that would cause an outage; how do you investigate whether it's a provider bug, a moved resource, or a real drift, before anyone hits apply?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07387_You_use_Terraform_in_the_deploy_pipeline_and_a_plan_shows_a_.md>)
- [Deep] [A deploy uses a distributed lock so only one rollout runs at a time, but a crashed job left the lock held and blocked all deploys; how would you make deploy locks self-healing without risking two concurrent rollouts?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07482_A_deploy_uses_a_distributed_lock_so_only_one_rollout_runs_at.md>)

## Related concepts

- [Pipeline Design Patterns](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*
- [Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*

