---
id: "TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "Python", "dbt"]
---

# Automated Dependency Updates

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [Python](<../../Python/_index.md>)
- [dbt](<../../dbt/_index.md>)

## Practice questions

- [Basic] [Why might a data team review the SQL and config of a pipeline change in a pull request even though the actual job runs on a managed Airflow or warehouse cluster?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27971_Why_might_a_data_team_review_the_SQL_and_config_of_a_pipelin.md>)
- [Intermediate] [You inherited a database repo with 150 stale branches, many holding half-finished migrations that reuse the same version numbers. How would you audit them, decide what to prune safely, and automate branch hygiene so version collisions stop recurring?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27818_You_inherited_a_database_repo_with_150_stale_branches_many_h.md>)
- [Deep] [Your team runs trunk-based development and two migrations with adjacent timestamps merged out of order, so Flyway applied them in a sequence that violated a foreign key on a busy table. How would you redesign the branching and migration-numbering scheme so ordering is deterministic regardless of merge order?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27779_Your_team_runs_trunk-based_development_and_two_migrations_wi.md>)
- [Deep] [A migration was approved that adds a foreign key without first indexing the referencing column, which will slow every write. What review checklist and automated CI rule would you put in place to catch missing-supporting-index patterns on FK migrations?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27862_A_migration_was_approved_that_adds_a_foreign_key_without_fir.md>)
- [Deep] [Your agent's tool definitions (JSON schemas) live in Git and a schema PR merged that broke the model's tool-calling. How would a required contract-validation check on the PR have caught it before merge?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27371_Your_agent_s_tool_definitions_JSON_schemas_live_in_Git_and_a.md>)
- [Deep] [Design an automated backport bot: when an alert-rule PR merges to main with a backport/stable label, it cherry-picks to the release branch and opens a PR. What conflict, ordering, and rule-dependency issues must it handle so it never silently drops a suppression?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27633_Design_an_automated_backport_bot_when_an_alert-rule_PR_merge.md>)
- [Deep] [A Kafka Connect connector config and a Flink job both live in Git, and a merged PR changed a partition key in one but not the other, causing serving skew. How would you couple the two configs in one atomic commit and add a train/serve-style parity check in CI?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27999_A_Kafka_Connect_connector_config_and_a_Flink_job_both_live_i.md>)
- [Deep] [You need to migrate the default branch of a payments service from master to main across CI, protection rules, open PRs, deploy pipelines, and every clone, with zero broken deploys. Sequence the steps.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28392_You_need_to_migrate_the_default_branch_of_a_payments_service.md>)

## Related concepts

- [Expand and Contract Migrations](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>) *(CI CD)*
- [Pull Request Pipelines](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>) *(CI CD)*
- [Schema Migration Strategy](<../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*

