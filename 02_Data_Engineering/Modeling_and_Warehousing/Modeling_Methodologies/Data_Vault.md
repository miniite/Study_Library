---
id: "02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault"
level: 5
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies"
tools: ["HashiCorp_Vault"]
---

# Data Vault

**L5 Expert** · [↑ Modeling Methodologies](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Data Vault stores enterprise data in three table types — hubs (business keys), links (relationships) and satellites (descriptive history) — so new sources and changes are added without reworking existing tables, and every record's load time and source are auditable.

### 1. The big picture
Kimball stars are optimised for querying; third normal form is optimised for integrity. Both struggle in a particular situation: many source systems, frequent schema changes, and a need to prove exactly what was loaded and when. Data Vault (created by Dan Linstedt) targets that situation. It separates three concerns that other models mix: **identity** (business keys), **relationships** (how keys associate) and **context** (descriptive attributes and their history).

Because each of these is stored in its own insert-only table type, a new source adds new satellites rather than altering existing structures; a new relationship adds a link; history is inherent. The cost is many tables and many joins, so Data Vault is almost never the query layer — it is the integration and audit layer, with dimensional stars (information marts) built on top for consumption.

### 2. Core concepts and vocabulary
- **Hub** — one row per distinct business key (customer number, product code) with a hash key, the business key, load datetime, and record source. No descriptive attributes.
- **Link** — one row per unique relationship between hubs (customer–order–product), holding the hub hash keys, a link hash key, load datetime, record source. Many-to-many by nature.
- **Satellite** — descriptive attributes of a hub or link, versioned over time; key is (parent hash key, load datetime), with hash diff and record source.
- **Hash key** — deterministic hash (MD5/SHA-1/SHA-256) of the business key(s), used as the primary key; enables parallel loading with no lookups.
- **Hash diff** — hash of a satellite row's attributes to detect change.
- **Raw Vault** — data stored as delivered, with no business rules applied (only hard rules: types, hashing).
- **Business Vault** — derived/cleaned data with business rules applied (computed satellites, PIT and bridge tables).
- **Point-in-time (PIT) table** — precomputed table that records, per hub key and snapshot date, which satellite version was valid, to avoid expensive range joins.
- **Bridge table** — precomputed set of link walks to speed queries.
- **Information mart** — star schema or flat views built from the vault for consumers.
- **Effectivity satellite** — tracks when a link relationship was active.
- **Record source / load date** — mandatory audit columns on every table.

### 3. How it works, step by step
**Modeling**
1. Identify core business concepts and their business keys → hubs.
2. Identify relationships and transactions between concepts → links.
3. Group the remaining attributes by source and change rate → satellites attached to hubs or links (e.g. `sat_customer_crm`, `sat_customer_billing`).

**Loading (raw vault)**
1. Land the source data in staging; compute hash keys and hash diffs, add load datetime and record source.
2. Insert new business keys into hubs (insert if the hash key is not present).
3. Insert new relationships into links.
4. Insert a new satellite row only when the hash diff differs from the latest row for that key.
5. All loads are insert-only, parallelisable and idempotent; hubs, links and satellites can load simultaneously because hash keys need no lookup.

**Consuming**
1. Business rules produce business vault objects (and PIT/bridge tables).
2. ETL builds information marts: dimensions from hub + satellites (+ PIT), facts from links + transactional satellites.

| Aspect | Data Vault | Kimball star | 3NF (Inmon) |
|---|---|---|---|
| Primary purpose | integration, history, audit | query/presentation | integration/integrity |
| Query friendliness | low (many joins) | high | medium |
| Handles schema change | excellent (add satellites) | needs redesign | moderate |
| History | built-in, insert-only | via SCD design | optional |
| Parallel loading | excellent | dependent order | dependent order |
| Table count | high | low | medium |

### 4. Worked example
```sql
CREATE TABLE hub_customer (
    hk_customer  binary(32) PRIMARY KEY,         -- SHA-256 of business key
    customer_id  varchar(30) NOT NULL,
    load_dts     datetime2 NOT NULL,
    rec_src      varchar(30) NOT NULL
);
CREATE TABLE hub_product (hk_product binary(32) PRIMARY KEY, sku varchar(30) NOT NULL, load_dts datetime2 NOT NULL, rec_src varchar(30) NOT NULL);

CREATE TABLE link_sale (
    hk_sale_link binary(32) PRIMARY KEY,         -- hash of customer_id || product_sku || order_no
    hk_customer  binary(32) NOT NULL,
    hk_product   binary(32) NOT NULL,
    order_no     varchar(20) NOT NULL,
    load_dts     datetime2 NOT NULL, rec_src varchar(30) NOT NULL
);

CREATE TABLE sat_customer_crm (
    hk_customer  binary(32) NOT NULL,
    load_dts     datetime2  NOT NULL,
    hash_diff    binary(32) NOT NULL,
    name varchar(100), city varchar(60), segment varchar(20),
    rec_src varchar(30) NOT NULL,
    PRIMARY KEY (hk_customer, load_dts)
);

-- insert only changed satellite rows
INSERT sat_customer_crm (hk_customer, load_dts, hash_diff, name, city, segment, rec_src)
SELECT s.hk_customer, SYSUTCDATETIME(), s.hash_diff, s.name, s.city, s.segment, 'CRM'
FROM stg_customer s
LEFT JOIN (SELECT hk_customer, hash_diff,
                  ROW_NUMBER() OVER (PARTITION BY hk_customer ORDER BY load_dts DESC) rn
           FROM sat_customer_crm) cur ON cur.hk_customer = s.hk_customer AND cur.rn = 1
WHERE cur.hk_customer IS NULL OR cur.hash_diff <> s.hash_diff;
```

Turning the vault into a dimension for consumers: a view that joins `hub_customer` to the satellite, with `LEAD(load_dts)` computing `valid_to`, yields an SCD Type 2 dimension; a PIT table avoids per-query window functions when several satellites must be aligned.

Choosing between "just land everything as SCD2 dimensions from CDC" and a Vault: at 200 sources, duplicate meaning of "customer" across systems, differing keys and frequent attribute changes make direct SCD2 dimensions brittle — each source change forces dimension redesign and reload, and there's no raw audit trail. The vault absorbs those changes and keeps the raw history needed for re-deriving marts.

### 5. Under the hood
Hash keys replace sequence lookups: because `hk = HASH(business key)` can be computed anywhere, a stream, a batch job and a Spark job can load hubs, links and satellites independently with no coordination, and rows can be inserted in any order (a link may arrive before its hub row is loaded — referential integrity is eventually consistent). Collisions are theoretically possible; SHA-256 or longer keys and key-hash standards (consistent trimming, case-folding, delimiter, NULL handling) are required — a normalisation mismatch silently creates two hubs for one key.

Satellites are append-only with a hash diff; "current" state is the latest row per key. This design suits immutable object storage and Delta/Iceberg tables and makes deletion semantics explicit (a deletion/tombstone flag or status satellite). Auditing becomes a query: what was known about customer X as of date Y.

The price shows in reads: reconstructing one customer entity needs hub ⨝ satellites with range logic; relationships need link ⨝ hubs. PIT and bridge tables materialise those joins; information marts hide them from consumers entirely. In lakehouses the open table formats' time travel and MERGE can overlap with some vault benefits, so evaluate whether the auditability and multi-source integration guarantees are really required, or whether a simpler medallion + dimensional design is sufficient.

### 6. Performance and design implications
- Loads are fast and parallel; queries on the raw vault are slow without PIT/bridge or marts.
- Table counts grow quickly (a single entity → one hub + several satellites + links); automate generation from metadata, don't hand-write.
- Hash computation standards must be centrally defined and tested.
- Storage grows with every change (insert-only), mitigated by columnar compression and hash-diff gating.
- Governance overhead: modeling skills are scarce; use only where the volatility/audit need justifies it.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `HASHBYTES('SHA2_256', CONCAT_WS('||', UPPER(TRIM(col))...))` | standard hash key | hubs/links |
| Hash diff over attribute columns | change detection | satellite loading |
| `LEAD(load_dts) OVER (PARTITION BY hk ORDER BY load_dts)` | derive validity end | building SCD2 views |
| PIT table build job | aligns satellites by snapshot date | mart query performance |
| Automation frameworks (dbtvault/AutomateDV, WhereScape) | generate vault code | large vaults |
| Delta `MERGE` / insert-only append | lakehouse load | satellites on lakehouse |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Many sources, frequent source changes, strict audit/lineage requirements | A few stable sources and a small team |
| Need parallel/streaming loads of independent feeds | Users query the vault directly with BI tools |
| Regulated environments needing reproducible history | Time-to-first-report is the top priority |
| Vault as integration layer + stars as presentation | Budget cannot support two layers |

### 9. Common mistakes
- Querying the raw vault directly → slow, hard SQL → build information marts/PIT tables.
- Inconsistent hash normalisation (case, trim, delimiters, NULLs) → duplicate hub rows → shared hashing standard and tests.
- Putting descriptive attributes in hubs → breaks the separation → satellites only.
- Applying business rules in the raw vault → can't re-derive from truth → rules in the business vault.
- Over-splitting satellites → excessive joins → group by source and rate of change, not per column.

### 10. 🎯 Interview kit
**30-second answer:** Data Vault models data as hubs (business keys), links (relationships) and satellites (historical attributes). It's insert-only, hash-keyed and metadata-driven, which makes integration of many changing sources and auditing easy; you then publish dimensional marts on top for querying.

**Structure a longer answer:** problem it solves → three table types → loading with hash keys → raw vs business vault, PIT/bridge → trade-offs vs Kimball/3NF → scenario.

**Likely follow-ups:**
- *Why not just SCD2 dimensions from CDC?* → breaks at scale with many sources and changes; no raw audit, redesign per change.
- *How do you know the business needs a vault?* → regulatory/audit needs, source volatility, number of sources; not because it's fashionable.
- *What are PIT tables for?* → avoid expensive range joins across satellites.
- *Where does it fit in a lakehouse?* → silver/integration layer; table formats provide time travel but not the integration model.

**Signals interviewers look for:** balanced view (not a silver bullet), correct hub/link/satellite roles, hash-key discipline, and the star layer on top.

### 11. Check yourself
- [ ] Model customers, products and orders as hubs, links and satellites.
- [ ] Explain how hash keys enable parallel loading.
- [ ] Write the satellite load that inserts only changed rows.
- [ ] Describe PIT tables and why they exist.
- [ ] Argue when you would not use Data Vault.
<!-- explainer:end -->

## Implementations (tools)

- [HashiCorp Vault](<../../../TOOLS/HashiCorp_Vault/_index.md>)

## Practice questions

- [Deep] [When would you argue that a fully normalized Inmon-style CIF is the wrong architectural bet even for a large regulated enterprise, and what evidence would you gather to prove it?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Q-12645_When_would_you_argue_that_a_fully_normalized_Inmon-style_CIF.md>)
- [Deep] [Describe how you'd design a mini-dimension plus outrigger to control SCD Type 2 growth for a customer demographics attribute set, and when it's overengineering.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Q-12667_Describe_how_you_d_design_a_mini-dimension_plus_outrigger_to.md>)
- [Deep] [A factless fact table is proposed to model coverage/eligibility. When is that elegant versus a sign someone doesn't understand the actual event grain?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Q-12691_A_factless_fact_table_is_proposed_to_model_coverage_eligibil.md>)
- [Deep] [A startup is scaling fast and asks whether to invest in a Vault now or defer. How do you frame the option value of modeling discipline versus premature rigor?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Q-12714_A_startup_is_scaling_fast_and_asks_whether_to_invest_in_a_Va.md>)
- [Deep] [How do you handle modeling of soft-deletes, re-activations, and tombstones across SCD dimensions and Vault satellites so history reflects reality, not source quirks?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Q-12734_How_do_you_handle_modeling_of_soft-deletes_re-activations_an.md>)
- [Expert] [How do you handle hash collisions and hash-diff drift in a Data Vault at trillions of rows, and what's your governance policy when a hashing algorithm must change?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Q-12661_How_do_you_handle_hash_collisions_and_hash-diff_drift_in_a_D.md>)
- [Expert] [Build-vs-buy: a vendor offers an automated Data Vault generation tool. How do you evaluate whether it earns its lock-in versus hand-rolled dbt macros?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Q-12680_Build-vs-buy_a_vendor_offers_an_automated_Data_Vault_generat.md>)
- [Expert] [You're modeling for a multi-region, multi-active data platform. How does the modeling paradigm interact with your replication and DR strategy?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Q-12711_You_re_modeling_for_a_multi-region_multi-active_data_platfor.md>)

## Related concepts

- [Star Schemas and Dimension Keys](<../Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>) *(Dimensional Modeling)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*
- [Kimball Approach](<../Dimensional_Modeling/Kimball_Approach.md>) *(Dimensional Modeling)*
- [Choosing a Database for the Workload](<../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Rollups vs On-demand Queries](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>) *(Data Architecture)*
- [Distinguishing Drift from Pipeline Breaks](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>) *(Data Architecture)*

## References

- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — The Big Picture Designing and Implementing a Lakehouse Platform › Data modeling considerations
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — The Big Picture Designing and Implementing a Lakehouse Platform › Data Vault modeling
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Lakehouse in the Real World › Data Modeling
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › Concrete example
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › Related topics
- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Glossary
- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Related topics
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Engineering Described › The 2020s: Engineering for the data lifecycle
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Major Undercurrents Across the Data Engineering Lifecycle
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Data modeling and design
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Modeling
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Engineering's Role › Distributed topology model
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Reference Architectures › Structure
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Reference Architectures › Centralized analytics topology model
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Reference Architectures › Centralized DE topology model
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Why the Industry Now Needs Data Contracts › Problems in Paradise
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Challenges of Scaling Data Infrastructure › How Data Drifts from Established Business Logic
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Change Management The Crux of People, Process, and Technology › Synthesize Your Findings
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Creating Your First Wins with Data Contracts › Map Out Where Key Stakeholders Fit in the Data Lifecycle
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Chapter 1. The Evolution of Data Architecture
- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › 3NF and Data Vault
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Creation of the Semantic Model
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Introduction to data product guidelines

