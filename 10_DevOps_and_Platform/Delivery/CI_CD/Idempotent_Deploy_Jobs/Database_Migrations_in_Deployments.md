---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Database_Migrations_in_Deployments"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Kubernetes", "AWS", "Git_and_GitHub", "GitLab"]
---

# Database Migrations in Deployments

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [GitLab](<../../../../TOOLS/GitLab/_index.md>)

## Practice questions

- [Intermediate] [How would you make a pipeline enforce that database migrations are reviewed by a DBA/senior for tables above a size threshold, while letting small tables auto-proceed, purely through automation?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07023_How_would_you_make_a_pipeline_enforce_that_database_migratio.md>)
- [Deep] [How would you design a deploy so that a failed migration can be detected and the migration rolled back before the new app version starts receiving traffic, rather than after?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07001_How_would_you_design_a_deploy_so_that_a_failed_migration_can.md>)
- [Deep] [You need a deploy to run a schema migration and the app pod that runs it as an init job races with a second replica that also tries to migrate; how would you guarantee exactly one migration runs per deploy?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07398_You_need_a_deploy_to_run_a_schema_migration_and_the_app_pod_.md>)
- [Deep] [How do you architect a pipeline so that developers get a genuinely faithful local pre-flight that catches 90% of CI failures, without maintaining two divergent definitions of the build that inevitably drift apart?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07192_How_do_you_architect_a_pipeline_so_that_developers_get_a_gen.md>)
- [Expert] [How do you handle a pipeline that deploys to multiple regions?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07448_How_do_you_handle_a_pipeline_that_deploys_to_multiple_region.md>)
- [Expert] [How would you architect a deploy for a sharded database service where each shard must be migrated and validated independently, and a failure on one shard shouldn't block the others?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06986_How_would_you_architect_a_deploy_for_a_sharded_database_serv.md>)
- [Expert] [You need to deploy a serialization-format change for cached objects, but old and new pods coexist during rollout and hit deserialization errors; how would you sequence the format change safely across a rolling deploy?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07430_You_need_to_deploy_a_serialization-format_change_for_cached_.md>)
- [Expert] [A deploy needs to atomically flip both a service version and its downstream service-mesh routing weights; how do you architect the coordination so the mesh and the CD system never disagree about the intended state?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07163_A_deploy_needs_to_atomically_flip_both_a_service_version_and.md>)

## Related concepts

- [Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*
- [Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>) *(Cloud Fundamentals)*

