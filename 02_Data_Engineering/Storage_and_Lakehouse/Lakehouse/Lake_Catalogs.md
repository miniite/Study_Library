---
id: "02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lake_Catalogs"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Lakehouse"
tools: ["AWS", "Databricks", "Hadoop"]
---

# Lake Catalogs

**L4 Advanced** · [↑ Lakehouse](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A lake catalog is the metadata service that turns piles of files into discoverable, named, governed tables — it tells every engine what exists, where it is, what its schema is, and who may touch it.

### 1. The big picture
The storage layer of a lakehouse holds all the data, but files alone are not usable: nobody can search them, engines don't know which files form a table or what its schema is, and nothing records who owns or may read what. A **data catalog** supplies that missing layer. It is where users search, explore and discover data, and where the platform applies unified governance, permissions, lineage and sharing.

Three related ideas are easy to confuse:
- **Metadata** — data about data: names, schemas, partitions, locations, owners, statistics, lineage.
- **Metastore** — the technical service engines query to resolve table names to locations and schemas (e.g., the Hive Metastore API).
- **Data catalog** — the user- and governance-facing layer built on top of metadata: search, descriptions, classification, lineage, access policies.

In a lakehouse the catalog is more central than in a traditional warehouse (where the database engine *is* the catalog) because multiple independent engines — Spark, Trino, Athena, Redshift Spectrum, EMR — need a *shared* definition of the same tables. One catalog gives them a unified view of metadata.

### 2. Core concepts and vocabulary
- **Technical metadata** — schemas, data types, partitions, file locations, statistics.
- **Business metadata** — descriptions, owners, glossary terms, domains, quality scores.
- **Operational metadata** — job runs, freshness, volumes, lineage.
- **Metastore** — service implementing table/partition lookup; Hive Metastore (HMS) is the de-facto API.
- **Catalog hierarchy** — typically catalog → schema/database → table → column.
- **Table format catalog** — for Iceberg/Delta, the pointer to the current metadata/log file location (e.g., REST catalog, Glue, Unity Catalog, Nessie).
- **Schema discovery / crawler** — automated scanning of files to infer schema and register tables (AWS Glue crawlers).
- **Fine-grained access control** — column/row-level policies, tag-based access, integrated with IAM.
- **Lineage** — which datasets/jobs produced a dataset (e.g., OpenLineage Dataset, Job, Run).
- **Data discovery** — search/browse with descriptions and tags.

### 3. How it works, step by step
**How an engine uses the catalog to answer a query** — `SELECT * FROM sales.orders WHERE order_date = '2026-03-01'`:

1. The engine asks the catalog for `sales.orders`.
2. The catalog returns: table format (Iceberg/Delta/Hive), schema, partition spec, location, and (for modern formats) the current metadata/snapshot pointer.
3. The catalog (or governance layer) checks the caller's permissions, applies row/column policies.
4. The engine reads the table's own metadata (manifests/log) and prunes files using stats.
5. It reads only the required Parquet files and columns from object storage.
6. Lineage/audit events are emitted if configured.

**How a table gets into the catalog**
1. *Explicit DDL* (`CREATE TABLE`) by an engine registers metadata.
2. *Crawlers* scan a path, infer schema and partitions and create/update catalog entries (schema versions tracked).
3. *Pipelines* register tables as part of deployment (infrastructure as code).
4. Business context (owners, descriptions, tags) is added by people or automation.

| Catalog option | Where it lives | Notes |
|---|---|---|
| Hive Metastore | Self-managed / EMR / Databricks legacy | De-facto API; relational backing store |
| AWS Glue Data Catalog | AWS | Serverless; crawlers; Athena/Redshift Spectrum/EMR integration; HMS-compatible API; IAM integration; fewer advanced governance features than specialist tools |
| Unity Catalog | Databricks | Unified governance, lineage, 3-level namespace |
| Iceberg REST / Nessie | Open | Engine-neutral table catalogs, git-like branching (Nessie) |
| Enterprise catalogs (Collibra, Alation, DataHub, Purview) | Various | Rich business/governance layer over technical catalogs |

### 4. Worked example
Registering and discovering a lakehouse table through AWS Glue + Athena.

```sql
-- Athena DDL registers an Iceberg table in the Glue Data Catalog
CREATE TABLE lakehouse.sales_orders (
  order_id   bigint,
  customer_id bigint,
  order_ts   timestamp,
  total      decimal(12,2)
)
PARTITIONED BY (day(order_ts))
LOCATION 's3://acme-lake/silver/sales_orders/'
TBLPROPERTIES ('table_type'='ICEBERG', 'format'='parquet');

-- Another engine (Spark on EMR) sees the same table through the same catalog
spark.sql("SELECT count(*) FROM glue_catalog.lakehouse.sales_orders")
```
Governance: grant a role read access to non-PII columns only (Lake Formation column-level permission), then verify with a user in that role: `SELECT customer_email …` fails; `SELECT order_id, total …` succeeds.

Reading the catalog: `SHOW CREATE TABLE`, `DESCRIBE FORMATTED`, or the Glue console lists schema versions, partitions and table properties. A schema-version history showing frequent flips indicates unstable upstream schemas worth a data contract.

### 5. Under the hood
- **Pointer semantics.** For open table formats the catalog's critical job is an **atomic pointer swap** to the new metadata file on commit. That atomicity gives concurrent writers safe optimistic concurrency; catalogs without atomic compare-and-swap (plain object-store listing) can't give true ACID.
- **HMS limitations.** Partition listing for tables with millions of partitions is slow in HMS; modern table formats keep partition/file info in their own manifests so the catalog stays small and fast.
- **Crawlers are convenient but risky.** They infer types from samples and can flip schemas as data varies; prefer explicit DDL/contracts for curated layers and use crawlers for discovery of raw zones.
- **Catalog vs governance.** A technical catalog (Glue) can be limited for advanced governance — data classification, stewardship workflows, cross-platform lineage — hence enterprise catalogs layered on top.
- **Catalogs can drown.** In big lakes or data meshes, thousands of undocumented tables make a catalog a graveyard; without ownership, descriptions, quality signals and deprecation, discovery fails. Treat metadata as a product with owners.
- **Lineage standards.** Open lineage events (Dataset/Job/Run with extensible facets) let schedulers, engines and catalogs share a common model for impact analysis.

### 6. Performance and design implications
- Catalog latency sits in every query's planning; keep partition counts sane and rely on table-format metadata for pruning.
- Use one authoritative catalog per environment; avoid duplicated/diverging copies of table definitions.
- Governance policies at the catalog layer apply to all engines; policies configured per-engine drift.
- Cross-account/region sharing and open protocols (Delta Sharing, Iceberg REST) reduce copies.
- Plan for catalog HA and backups — it's a control-plane dependency of every job.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE … LOCATION … TBLPROPERTIES` | Registers a table | Explicit, reviewed registration |
| `SHOW TABLES`, `DESCRIBE FORMATTED` | Inspect catalog entries | Discovery/debugging |
| `MSCK REPAIR TABLE` / `ALTER TABLE ADD PARTITION` | Sync Hive-style partitions | Legacy Hive tables |
| Glue crawler schedule + schema-change policy | Auto-discover and update | Raw zone exploration |
| Lake Formation / Unity Catalog grants | Column/row-level access | Governance |
| Table `COMMENT` / tags | Business metadata | Searchability |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Managed catalog (Glue, Unity): small team, cloud-native | Need engine-neutral portability across clouds |
| Open REST/Nessie: multi-engine, avoid lock-in | You lack ops capacity to run it |
| Enterprise catalog on top: stewardship, glossary, compliance | Tooling sprawl; unused metadata |
| Crawlers: discover raw data fast | Curated layers needing stable schemas |

### 9. Common mistakes
- Registering tables by hand in each engine → divergent definitions → single shared catalog.
- Relying on crawlers for curated schemas → surprise type changes → explicit DDL + contracts.
- No ownership/description metadata → undiscoverable "swamp" → require owner + description at creation.
- Treating the catalog as optional for security → inconsistent access → centralize policies.
- Ignoring catalog availability → all jobs fail when it's down → HA and backups.

### 10. 🎯 Interview kit
**30-second answer:** A lake catalog is the metadata layer that maps table names to files, schemas, partitions and policies so multiple engines see the same tables. It combines a technical metastore (resolve table → location/schema/snapshot) with discovery and governance (descriptions, ownership, permissions, lineage). Options include Hive Metastore, AWS Glue, Unity Catalog and open Iceberg catalogs.

**Structure a longer answer:** why files alone fail → metadata vs metastore vs catalog → query-time flow → options and trade-offs → governance and lineage → pitfalls.

**Likely follow-ups:**
- *Glue Data Catalog strengths/limits?* → Serverless, integrated with AWS analytics, HMS-compatible, IAM; limited advanced governance/lineage vs specialist tools.
- *Why does a table format need an atomic catalog operation?* → To swap the current-metadata pointer safely under concurrent commits.
- *Crawler vs explicit DDL?* → Discovery vs stability.
- *How avoid a catalog "swamp"?* → Ownership, descriptions, quality/usage signals, deprecation.

**Signals interviewers look for:** distinction between metastore/catalog, understanding of commit atomicity and governance, realism about discovery.

### 11. Check yourself
- [ ] Distinguish metadata, metastore and data catalog.
- [ ] Trace a query through the catalog to the Parquet files.
- [ ] Compare Glue, Hive Metastore and an Iceberg REST catalog.
- [ ] Explain why crawlers are risky for curated tables.
- [ ] Design minimum metadata requirements for every new table.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Databricks](<../../../TOOLS/Databricks/_index.md>)
- [Hadoop](<../../../TOOLS/Hadoop/_index.md>)

## Practice questions

- [Basic] [What are the medallion architecture layers (bronze silver gold) and what does each hold?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11849_What_are_the_medallion_architecture_layers_bronze_silver_gol.md>)
- [Basic] [What role does a metadata catalog play in a data lake?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11867_What_role_does_a_metadata_catalog_play_in_a_data_lake.md>)
- [Basic] [What is the AWS Glue Data Catalog and what does it store?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11868_What_is_the_AWS_Glue_Data_Catalog_and_what_does_it_store.md>)
- [Basic] [What is Amazon Athena and how does it query data in S3?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11872_What_is_Amazon_Athena_and_how_does_it_query_data_in_S3.md>)
- [Basic] [What does the bronze layer typically store in medallion architecture?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11878_What_does_the_bronze_layer_typically_store_in_medallion_arch.md>)
- [Basic] [What is Amazon S3 and why is it a common backbone for data lakes?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11909_What_is_Amazon_S3_and_why_is_it_a_common_backbone_for_data_l.md>)
- [Intermediate] [What is the Hive Metastore and why is it still relevant in modern lakes?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11869_What_is_the_Hive_Metastore_and_why_is_it_still_relevant_in_m.md>)
- [Intermediate] [What is Unity Catalog in Databricks and what does it govern?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11870_What_is_Unity_Catalog_in_Databricks_and_what_does_it_govern.md>)

## Related concepts

- [Cataloging External Data](<../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Cataloging_External_Data.md>) *(Catalog and Metadata)*
- [Tiered Lake Storage](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>) *(Data Architecture)*
- [Data Warehouse Fundamentals](<../../Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*

## References

- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › Common services
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Data Catalogs › Chapter 4. Data Catalogs
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Data Catalogs › Features of a Data Catalog
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Data Catalogs › Data Governance and Security
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Governance and cataloguing › Aws Glue Data Catalog
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Governance and cataloguing › Google Data Cloud Catalog
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Governance and cataloguing › Erwin Data Catalog
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Governance and cataloguing › Other technologies
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Designing a Data Catalog
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Building a Data Catalog
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Democratizing Data Quality › Prioritizing a Data Catalog
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Democratizing Data Quality › Third-party
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Chapter 3. Overview of AWS Analytics and Auxiliary Services
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › AWS Glue
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Amazon Athena
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Reference Architecture: Lakehouse with Glue, Redshift, and Athena
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Managed tables
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Implementing Data Governance › Governing Data with Unity Catalog
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Implementing Data Governance › Unity Catalog Architecture
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Implementing Data Governance › UC Three-Level Namespace
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Data Contract Components Data Assets and Contract Definition › Data catalogs
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Component C: Detection

