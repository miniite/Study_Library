---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Multi-tenant_Migrations"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: ["AWS", "Apache_Kafka", "Redis"]
---

# Multi-tenant Migrations

**L5 Expert** · [↑ System Design Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Design a system to merge a guest cart with the user's cart after they log in.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42341_Design_a_system_to_merge_a_guest_cart_with_the_user_s_cart_a.md>)
- [Deep] [Design a distributed cache with a write-through vs write-around vs write-back policy for a hot product-inventory key. Which do you ship for correctness-critical stock, and why?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42174_Design_a_distributed_cache_with_a_write-through_vs_write-aro.md>)
- [Deep] [Design an outbox + relay so domain events are published atomically with the DB commit. How does the relay guarantee at-least-once to Kafka, and how do downstream consumers dedupe?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42506_Design_an_outbox_relay_so_domain_events_are_published_atomic.md>)
- [Deep] [Design a per-tenant data export + import for enterprise customers migrating in and out. How do you keep the export consistent while data changes underneath, and stream large volumes without OOM?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42565_Design_a_per-tenant_data_export_import_for_enterprise_custom.md>)
- [Deep] [Design the architecture for a system where the frontend must support real-time multi-device continuity (start on phone, continue on laptop mid-flow) for a purchase. How do you sync ephemeral cart/session state across devices with low latency and conflict-free merges?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42745_Design_the_architecture_for_a_system_where_the_frontend_must.md>)
- [Expert] [Design the architecture for allowing customers to build their own dashboards/reports over your data (embedded analytics) without letting a badly-written customer query take down your OLTP or leak another tenant's rows. Where does the query execution boundary live?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42193_Design_the_architecture_for_allowing_customers_to_build_thei.md>)
- [Expert] [How do you architect for 'correctness under partial deploy' where, during a rolling release, old and new code run simultaneously and produce events consumed by both versions? Give the design rules that prevent silent data corruption.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42634_How_do_you_architect_for_correctness_under_partial_deploy_wh.md>)
- [Expert] [Design a system where product teams can define workflows (order fulfillment, KYC) as long-running orchestrations that survive process restarts and take days to complete. Would you build on a durable-execution engine or roll your own sagas, and why?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42655_Design_a_system_where_product_teams_can_define_workflows_ord.md>)

## Related concepts

- [Audit and Compliance Platforms](<../../High_Level_Design/Audit_and_Compliance_Platforms.md>) *(High Level Design)*
- [Two-level Cache Design](<../../Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*
- [Distributed Cache Design](<../../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Refactoring Data Access You'll recall we presented a design for the Reservation Service in Chapter

