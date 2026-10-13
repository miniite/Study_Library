---
id: "04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution"
tools: []
---

# Schema Changes Under Long Transactions

**L5 Expert** · [↑ Schema Evolution](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [A change advisory board is slowing your teams to a crawl. How do you replace human CAB gating for schema changes with automated policy without increasing incident rate? Show the metrics you'd commit to.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Q-38957_A_change_advisory_board_is_slowing_your_teams_to_a_crawl_How.md>)
- [Deep] [How do you manage index creation as part of schema evolution on hot tables — concurrent index builds, invalid-index cleanup, write amplification, and the decision to build during off-peak vs anytime?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Q-38982_How_do_you_manage_index_creation_as_part_of_schema_evolution.md>)
- [Deep] [How do you handle the situation where a migration must run inside application deploy CD but the DB change and code change have different rollback semantics? Describe your deploy-and-migrate ordering contract.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Q-39013_How_do_you_handle_the_situation_where_a_migration_must_run_i.md>)
- [Deep] [How do you evolve schema for wide-column stores (Cassandra/ScyllaDB) where you 'model per query' — what does adding a new access pattern require, and why is it closer to a data migration than a DDL?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Q-39027_How_do_you_evolve_schema_for_wide-column_stores_Cassandra_Sc.md>)
- [Expert] [Describe your approach to migrating a schema while simultaneously upgrading the database engine major version (e.g. Postgres 12 to 16). How do you decouple those two risk vectors or deliberately combine them?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Q-38945_Describe_your_approach_to_migrating_a_schema_while_simultane.md>)
- [Expert] [How do you evolve schema for a globally-distributed, multi-region active-active database (e.g. Spanner/CockroachDB/Yugabyte) where DDL is itself a distributed transaction? What new failure modes appear?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Q-38959_How_do_you_evolve_schema_for_a_globally-distributed_multi-re.md>)
- [Expert] [How do you evolve schemas while a heterogeneous CDC pipeline (Debezium) is streaming changes downstream, given that DDL events themselves flow through the pipeline? What breaks and how do you protect consumers?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Q-38979_How_do_you_evolve_schemas_while_a_heterogeneous_CDC_pipeline.md>)
- [Expert] [How do you evolve a schema that is replicated to edge/offline-first clients (mobile SQLite, PouchDB) where clients may sync a change made months ago? What conflict and versioning model do you enforce?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Q-39011_How_do_you_evolve_a_schema_that_is_replicated_to_edge_offlin.md>)

## Related concepts

- [Schema Migration Strategy](<../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*
- [Event Schema Evolution](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>) *(Distributed Systems Core)*

