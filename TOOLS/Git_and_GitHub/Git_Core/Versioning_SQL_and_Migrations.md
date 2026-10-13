---
id: "TOOLS/Git_and_GitHub/Git_Core/Versioning_SQL_and_Migrations"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Versioning SQL and Migrations

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Basic] [As a database developer, why would you keep your .sql schema and migration scripts in Git rather than only inside the database server?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27679_As_a_database_developer_why_would_you_keep_your_sql_schema_a.md>)
- [Basic] [You changed a column's data type in a CREATE TABLE script. How would you record that as a clear, self-contained Git commit?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27686_You_changed_a_column_s_data_type_in_a_CREATE_TABLE_script_Ho.md>)
- [Basic] [Why is it useful to commit a small script that populates lookup tables (like country codes) as versioned reference data?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27727_Why_is_it_useful_to_commit_a_small_script_that_populates_loo.md>)
- [Intermediate] [Why is it important that migration scripts, once merged and applied, are never edited in place but instead corrected by a new migration?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27702_Why_is_it_important_that_migration_scripts_once_merged_and_a.md>)
- [Intermediate] [Why is it useful to include both the 'up' migration and a 'down' or rollback script in the same commit?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27737_Why_is_it_useful_to_include_both_the_up_migration_and_a_down.md>)
- [Intermediate] [A migration idempotency question came up: a re-run of the deploy applied a CREATE INDEX twice and errored. How does the migration tool's Git-tracked history plus its DB tracking table prevent double-apply, and what breaks that guarantee if someone edits an applied file?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27864_A_migration_idempotency_question_came_up_a_re-run_of_the_dep.md>)
- [Deep] [Why is it important that the migration currently deployed to production always corresponds to a specific commit in Git?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27769_Why_is_it_important_that_the_migration_currently_deployed_to.md>)
- [Deep] [You need to prove which migration version is actually applied to a production database during an incident when all you have is the schema-history tracking table and the Git log. Walk through correlating the two and what can make them disagree.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27799_You_need_to_prove_which_migration_version_is_actually_applie.md>)

## Related concepts

- [Schema Migration Strategy](<../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*

