---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["AWS", "Apache_Kafka", "Redis"]
---

# Audit and Compliance Platforms

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Deep] [Design a distributed unique-ID generator (like Snowflake) for a system creating 1M IDs/sec across data centers — how do you keep IDs roughly time-ordered and collision-free?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29625_Design_a_distributed_unique-ID_generator_like_Snowflake_for_.md>)
- [Deep] [You're the reviewer on a design that puts all business logic in database stored procedures for 'performance' — probe it: what does that do to testability deploy-safety and horizontal scaling and how would you re-architect the hot path?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29899_You_re_the_reviewer_on_a_design_that_puts_all_business_logic.md>)
- [Expert] [You must architect data-residency compliance so EU and India user data never leaves-region while a global feed still works — how do you split storage, keep joins possible, and prove compliance to auditors?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29793_You_must_architect_data-residency_compliance_so_EU_and_India.md>)
- [Expert] [Design a data-deletion / right-to-be-forgotten architecture that must purge a user from 50 services, backups, caches, and analytics within a legal deadline — how do you prove completeness?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29823_Design_a_data-deletion_right-to-be-forgotten_architecture_th.md>)
- [Expert] [Design the multi-tenant data-partitioning strategy for a platform where tenant sizes span 6 orders of magnitude — how do you co-locate the millions of tiny tenants and give the handful of whales dedicated capacity?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29853_Design_the_multi-tenant_data-partitioning_strategy_for_a_pla.md>)
- [Expert] [You must architect how the platform handles a 10x cardinality explosion in a metrics/observability system that's now costing more than the product it monitors — bend the cost curve without going blind.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29865_You_must_architect_how_the_platform_handles_a_10x_cardinalit.md>)
- [Expert] [Design a per-tenant encryption-key architecture (bring-your-own-key) for a multi-tenant SaaS where an enterprise can revoke their key to instantly cut off access — how do you architect envelope encryption key caching and the crypto-shredding delete without killing read latency?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29910_Design_a_per-tenant_encryption-key_architecture_bring-your-o.md>)
- [Expert] [Design a right-to-be-forgotten deletion architecture that must purge a user across 60 services CDC streams backups and a warehouse within a legal deadline — how do you orchestrate the fan-out prove completeness and handle backups you can't selectively edit?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29937_Design_a_right-to-be-forgotten_deletion_architecture_that_mu.md>)

## Related concepts

- [Multi-tenant Migrations](<../System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Platform-level Data Quality](<../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Platform-level_Data_Quality.md>) *(Data Platform Strategy)*
- [Database Access Control and Auditing](<../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>) *(Security and Access)*

## References

- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 7Scaling Data Solutions
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture

