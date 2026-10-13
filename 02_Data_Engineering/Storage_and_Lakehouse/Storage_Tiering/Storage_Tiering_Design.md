---
id: "02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Storage_Tiering_Design"
level: 5
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering"
tools: ["AWS"]
---

# Storage Tiering Design

**L5 Expert** · [↑ Storage Tiering](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Storage tiering moves data between hot, warm, cold and archive storage as its access frequency falls, cutting cost while honouring latency, retention and legal requirements — but only if the boundaries, restore expectations and table-format metadata are designed together.

### 1. The big picture
Not all data is equal. New data is queried constantly and must be fast; last year's data is touched a few times a month; data kept for audit may never be read. Paying premium storage prices for all of it wastes money, and putting everything in the cheapest archive tier makes everyday analytics unusable. **Data retention policies** state how long data is kept and when it moves between tiers; **archiving strategies** cover long-term preservation of data that is no longer active but must be retained for compliance, audit or historical reference.

Tiering shows up at several levels:
- **Object storage classes** — Amazon S3 Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier Instant/Flexible Retrieval, Glacier Deep Archive; Azure Hot/Cool/Cold/Archive; GCS Standard/Nearline/Coldline/Archive.
- **Service-level tiers** — OpenSearch hot/UltraWarm/cold, Kafka/MSK broker tiered storage, warehouse long-term storage pricing.
- **Database partition tiers** — in SQL Server, partitions of a large table can live on different filegroups: new partitions on fast, expensive storage and old ones on slow, cheap storage, matched to SLAs.

The core skill is deciding tier boundaries from *measured access patterns* and *total cost* (storage + retrieval + request + minimum-duration charges), not from intuition.

### 2. Core concepts and vocabulary
- **Hot / warm / cold / archive** — descending access frequency, descending storage price, rising retrieval cost and latency.
- **Lifecycle rule** — policy that transitions or expires objects by age, prefix, tag or size.
- **Retrieval latency** — milliseconds (Instant), minutes to hours (Flexible), up to ~12–48 hours (Deep Archive).
- **Minimum storage duration** — billing floor (e.g., 30, 90, 180 days) for IA/Glacier tiers even if deleted earlier.
- **Transition request cost** — per-object charge for moving objects; punishing for millions of tiny files.
- **Retention lock / legal hold** — immutability controls that must override cost-driven expiry.
- **Restore SLA** — agreed time and cost for recovering archived data.
- **Intelligent-Tiering** — automatic movement by observed access, with a monitoring fee per object.
- **Object-level vs table/partition-level tiering** — policy on keys vs policy aligned to logical partitions.

### 3. How it works, step by step
**Designing a tier ladder:**
1. **Measure access.** From S3 access logs/inventory, query engine logs or catalog statistics, build the age-vs-read histogram per dataset (what % of reads touch data older than 7/30/90/365 days).
2. **Classify datasets** by access shape: steady-hot, age-decaying (most common), bursty/unpredictable, compliance-only.
3. **Set boundaries** where marginal retrieval + latency cost is lower than storage savings. Typical ladder: Standard (0–30/90 days) → Infrequent Access (to ~1 year) → Glacier Instant/Flexible (to retention) → Deep Archive (regulatory).
4. **Check fees:** minimum duration, per-object transition and retrieval fees, small-object overhead (objects below ~128 KB can cost more to transition than they save).
5. **Compact before tiering** so you transition fewer, larger objects.
6. **Express policy** as lifecycle rules by prefix/tag, aligned to partitions (e.g., `year=`/`month=` prefixes).
7. **Protect:** apply legal hold / object-lock exceptions and deletion safeguards.
8. **Define restore SLA** for each cold tier and test it.
9. **Review quarterly** against actual access and current prices.

### 4. Worked example
```json
// S3 lifecycle rule for a partitioned fact table, applied by prefix
{
  "Rules": [{
    "ID": "events-tiering",
    "Filter": {"And": {"Prefix": "lake/silver/events/", "ObjectSizeGreaterThan": 131072}},
    "Status": "Enabled",
    "Transitions": [
      {"Days": 90,  "StorageClass": "STANDARD_IA"},
      {"Days": 365, "StorageClass": "GLACIER_IR"}
    ],
    "NoncurrentVersionExpiration": {"NoncurrentDays": 30}
  }]
}
```
```sql
-- SQL Server: place old partitions on cheaper storage
CREATE PARTITION SCHEME ps_Sales AS PARTITION pf_SalesDate
TO (fg_Archive, fg_Slow, fg_Fast, fg_Fast);   -- oldest → newest
```
Reading the result: use S3 Storage Lens / inventory to confirm bytes per storage class; a monthly report on aged data will now *read from IA* (fine) or *Glacier Flexible* (needs restore) — know which before promising SLAs. If transition request charges exceed the storage saving, the rule is moving too many tiny objects: compact first or raise the size filter.

### 5. Under the hood
- **Table formats and tiering:** Delta/Iceberg/Hudi metadata records *paths* of data files. A lifecycle rule that moves objects to a class requiring restore (Glacier Flexible/Deep) makes those files unreadable until restored; the table still "points" at them and queries fail or hang. Instant-retrieval tiers (IA, Glacier IR) keep objects readable and are safe. Never *delete or rename* files from outside the table's maintenance (`VACUUM`, `expire_snapshots`), or the manifest references vanish.
- **Time travel interplay:** old snapshots keep old data files alive; retention for time travel affects what can be tiered or deleted. Reproducible ML datasets keep every snapshot but rarely re-read old ones: tier old snapshots' unique files to cold storage and keep manifests and recent snapshots hot.
- **Partition-level tiering:** because partitions are directories/prefixes, lifecycle by prefix lines up with partition boundaries; for tables partitioned by hour or tenant, rules per prefix become unwieldy — prefer table-level maintenance jobs that tag objects or move whole partitions.
- **BigQuery/Warehouse automatic pricing:** BigQuery charges lower "long-term" storage rates for table partitions unmodified for 90 days — no performance change. Exploit by not rewriting old partitions (avoid unnecessary updates/compaction touching cold partitions).
- **Legal holds:** object-lock retention and legal hold take precedence in the storage layer; lifecycle expiry cannot delete a locked object. Architecturally, define a hold registry and make lifecycle automation read it.
- **Minimum duration trap:** moving data into IA/Glacier and then deleting or promoting it within the minimum period still bills the full period; unpredictable access plus aggressive tiering produces surprise charges.

### 6. Performance and design implications
- Cold data in the same table as hot data means occasional queries scanning old partitions will be slow or fail; communicate restore SLAs and restrict default queries to recent partitions (partition filters, views).
- For bursty/unpredictable access, use Intelligent-Tiering (auto-moves; pay small monitoring fee) rather than age rules; avoid it for tiny objects.
- Egress and retrieval charges can exceed storage savings for data touched monthly; model total cost.
- Measure: bytes per tier, retrieval GB and cost, restore requests and wait time, transition request cost, percentage of queries touching cold partitions.
- Platform defaults: provide templates (hot 90 days → IA → archive per data class), guardrails (cannot archive datasets tagged "serving"), and approvals for changes — domain teams get sane defaults without the ability to archive critical data by accident.
- Feedback loop: schedule a job that recomputes age-access histograms and recommends boundary changes as usage and price lists change.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| S3 Lifecycle configuration | Transition/expire by age, prefix, tag | Default tiering |
| S3 Intelligent-Tiering | Automatic access-based tiering | Unpredictable access |
| S3 Object Lock / Legal Hold | Immutability | Compliance, legal hold |
| S3 Inventory / Storage Lens / access logs | Visibility into usage | Choose boundaries |
| Glacier restore request (Expedited/Standard/Bulk) | Temporary readable copy | Pulling archived data back |
| `VACUUM` / `expire_snapshots` | Table-aware deletion | Never delete files directly |
| Azure access tier / lifecycle policy | Hot/Cool/Archive moves | ADLS Gen2 lakes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Age-based lifecycle | Access is bursty and not age-correlated |
| Glacier Instant Retrieval | Data is never read — Deep Archive is cheaper |
| Deep Archive | Anything an analyst may need within hours |
| Intelligent-Tiering | Many tiny objects (monitoring fee) |
| Expire old data | Legal hold or lineage dependencies unknown |

### 9. Common mistakes
- "Everything to Deep Archive" → analytics break and restores cost more → tier by measured access with a restore SLA.
- Transitioning millions of tiny objects → request charges exceed savings → compact first / size filter.
- Moving data files out from under table metadata → broken queries → tier only by classes that remain readable, or manage via table maintenance.
- Lifecycle expiry without legal-hold awareness → compliance breach → object lock and hold registry override.
- Tiering then deleting early → minimum-duration charges → align retention with minimums.
- Set-and-forget policies → drift from access reality → feedback loop.

### 10. 🎯 Interview kit
**30-second answer:** I derive tier boundaries from measured age-vs-access data, compare total cost including retrieval, request and minimum-duration fees, and build a ladder like Standard → IA → Glacier/Deep Archive using lifecycle rules aligned to partitions. I make sure table metadata stays valid, legal holds override cost policies, and restore SLAs are explicit and tested.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Monthly report suddenly 6 hours?* → its data aged into a restore-required tier; check access/lifecycle logs; fix by instant tiers, keeping needed partitions hot, or restore SLA/pre-staging.
- *Transition costs dwarf savings?* → tiny objects; compact and filter by size.
- *Table format vs lifecycle?* → manifests reference paths; use readable tiers and table maintenance procedures.
- *Legal hold vs expiry?* → object lock/hold registry wins by design.
- *Restore SLA?* → agree latency, cost, bulk limits; enforce via request workflow.

**Signals interviewers look for:** quantified cost model, awareness of hidden fees, governance, operations.

### 11. Check yourself
- [ ] Build an age-vs-read histogram and place two boundaries from it.
- [ ] List four charges beyond per-GB storage price.
- [ ] Explain what breaks when a Glacier Flexible transition hits files referenced by an Iceberg manifest.
- [ ] Draft a restore SLA table for each tier.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [Walk me through how you would design a multi-tier storage strategy across S3 Standard / Infrequent-Access / Glacier for a petabyte-scale lakehouse, and how you decide the tier boundaries.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Q-41553_Walk_me_through_how_you_would_design_a_multi-tier_storage_st.md>)
- [Deep] [How do you tier feature-store and training datasets when reproducibility demands you keep every historical snapshot but most are never re-read?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Q-41588_How_do_you_tier_feature-store_and_training_datasets_when_rep.md>)
- [Deep] [BigQuery auto-drops table partitions to long-term storage after 90 days of no edits. How do you exploit or work around that behavior in a partitioning + tiering design?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Q-41625_BigQuery_auto-drops_table_partitions_to_long-term_storage_af.md>)
- [Expert] [How does Iceberg / Delta / Hudi metadata interact with lifecycle transitions, and what breaks if lifecycle rules move data files out from under the table's manifest?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Q-41561_How_does_Iceberg_Delta_Hudi_metadata_interact_with_lifecycle.md>)
- [Expert] [A dataset is under legal hold but your lifecycle rule tried to expire it. How do you architect lifecycle so legal holds and retention locks always win over cost policies?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Q-41579_A_dataset_is_under_legal_hold_but_your_lifecycle_rule_tried_.md>)
- [Expert] [How do you build tiering into a self-serve data platform so domain teams get sane defaults but can't accidentally archive data that needs to stay hot?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Q-41596_How_do_you_build_tiering_into_a_self-serve_data_platform_so_.md>)
- [Expert] [How do you architect a 'restore SLA' contract with downstream teams — what latency and cost do you commit to for pulling archived data back, and how do you enforce it?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Q-41617_How_do_you_architect_a_restore_SLA_contract_with_downstream_.md>)
- [Expert] [How do you design a feedback loop that continuously re-tunes tier boundaries as access patterns and prices change, rather than a set-and-forget policy?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Q-41632_How_do_you_design_a_feedback_loop_that_continuously_re-tunes.md>)

## Related concepts

- [Retention Costs and Compliance](<../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Retention_Costs_and_Compliance.md>) *(FinOps)*
- [Tiered Lake Storage](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>) *(Data Architecture)*
- [Table Format Internals](<../Table_Formats/Table_Format_Internals.md>) *(Table Formats)*
- [Small Files and Compaction](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>) *(Data Architecture)*
- [Lakehouse Fundamentals](<../Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Data Warehouse Fundamentals](<../../Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Enriching Data Catalogs with Data Classification › Selecting Storage Solutions for Hot and Cold Data
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Enriching Data Catalogs with Data Classification › Storage Tier Decisions for Different Access Patterns
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Defining Data Retention Policy and Archiving Strategies
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Defining Data Retention Policy and Archiving Strategies › Optimizing Data Management with Amazon S3
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination › Maintain Hot/Warm/Cold Data
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Beyond the Exam
- *Azure Data Fundamentals* — Michael John Pena — Azure SQL Services and Open Source Options › Burstable tier
- *Azure Data Fundamentals* — Michael John Pena — Azure Storage Solutions › Access Tiers and Cost Management
- *Azure Data Fundamentals* — Michael John Pena — Azure Storage Solutions › Cool tier: Balancing access and economy
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Backup and Recovery › Building Block 2: Tiered Storage
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Backup and Recovery › Online, low-performance storage
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Datastore Field Guide › Storage
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — A Data Architecture Sampler › Data Access Layer
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Understanding data access frequency
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Selecting a storage system
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Data lifecycle management
- *Microsoft Azure Storage: The Definitive Guide* — Avinash Valiramani — Storage components › Storage tiers
- *Microsoft Azure Storage: The Definitive Guide* — Avinash Valiramani — Networking endpoints › Storage access tiers
- *Microsoft Azure Storage: The Definitive Guide* — Avinash Valiramani — Default access tier configuration
- *Microsoft Azure Storage: The Definitive Guide* — Avinash Valiramani — Storage tiers for file shares

