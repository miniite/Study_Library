---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Federated_vs_Centralized_Data_Ownership"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Federated vs Centralized Data Ownership

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [You're the principal on a platform where teams keep choosing last-write-wins for convenience and silently losing data under clock skew. Design the guardrails, defaults, and review gates that make LWW a deliberate, justified choice rather than an accidental one.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23531_You_re_the_principal_on_a_platform_where_teams_keep_choosing.md>)
- [Deep] [You're a principal with no direct authority over three teams whose services must adopt a new idempotency-and-versioning standard. Explain how you make the paved road irresistible, handle the holdout team, and measure that adoption is real rather than a checkbox.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23577_You_re_a_principal_with_no_direct_authority_over_three_teams.md>)
- [Expert] [You must pick a datastore standard for the whole org: one blessed database vs teams choosing per-workload. Argue both sides at the platform level (paved road vs autonomy), and describe the governance model you'd actually ship so you get 80% of the benefit without becoming a bottleneck.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23319_You_must_pick_a_datastore_standard_for_the_whole_org_one_ble.md>)
- [Expert] [Design a three-year data architecture and governance strategy as the org grows from 5 to 50 services: ownership of shared entities, schema evolution, PII handling, a data contract mechanism between producers and consumers, and how you prevent the shared-database anti-pattern from creeping back in.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23340_Design_a_three-year_data_architecture_and_governance_strateg.md>)
- [Expert] [A dependency graph analysis reveals that 60% of your services transitively depend on one internal auth service. As the principal, is that a problem or a feature? Argue both sides, and if you'd de-risk it, how do you reduce the shared-fate coupling without shattering into inconsistent per-service auth?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23366_A_dependency_graph_analysis_reveals_that_60_of_your_services.md>)
- [Expert] [You're deciding data-mesh versus a centralized data platform as an org bet. When does federating data ownership to product teams pay off versus just spreading inconsistency, and what platform primitives (data contracts, discoverability, SLAs) must exist before you decentralize?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23604_You_re_deciding_data-mesh_versus_a_centralized_data_platform.md>)
- [Expert] [You're deciding whether the org standardizes on one blessed telemetry-agent (OTel Collector) everywhere or lets teams choose. Argue the paved-road-vs-autonomy trade-off for observability agents specifically (consistency of instrumentation, upgrade blast radius, operational load) and the governance you'd ship.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24018_You_re_deciding_whether_the_org_standardizes_on_one_blessed_.md>)
- [Expert] [You inherit a shared-database anti-pattern where 30 services reach into each other's tables. As principal, design the multi-year path to data ownership: the data-contract mechanism between producers and consumers, ownership of shared entities, and the review gates that stop it creeping back.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24199_You_inherit_a_shared-database_anti-pattern_where_30_services.md>)

## Related concepts

- [Platform-level Data Quality](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Platform-level_Data_Quality.md>) *(Data Platform Strategy)*
- [API Composition and BFF](<../../../System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>) *(System Design Core)*
- [Cross-domain Joins in a Mesh](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>) *(Data Architecture)*

## References

- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › CHAPTER 1Evolution of Data Architecture
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › Key considerations
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › Cautions and limits
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › Related topics
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Engineering Described › Upstream stakeholders
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Engineering Described › Conclusion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › "Good" Data Architecture
- *Building Medallion Architectures* — Piethein Strengholt — Case Study Amadeus, a Tech Leader in the Travel Industry › Data Governance
- *Building Medallion Architectures* — Piethein Strengholt — Case Study Strategic Data Transformation at ZEISS › Recommendations and Best Practices
- *Building Medallion Architectures* — Piethein Strengholt — Scaling the Medallion Architecture › Chapter 11. Scaling the Medallion Architecture
- *Building Medallion Architectures* — Piethein Strengholt — Scaling the Medallion Architecture › Decentralization of Data Management
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Architectural Patterns in Data Engineering › 3. Architectural Patterns in Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Future of Data Engineering › 15. Future of Data Engineering
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Why Data Quality Deserves Attention---Now › More specialized data teams
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Why Data Quality Deserves Attention---Now › Decentralized data teams
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Building End-to-End Lineage › Exercise "Controlled Freedom" When Dealing with Stakeholders
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Building End-to-End Lineage › Invest in a Decentralized Data Team
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Why the Industry Now Needs Data Contracts › Data Architecture in Disrepair
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Challenges of Scaling Data Infrastructure › Core Challenges for Modern Data Engineering Teams
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Challenges of Scaling Data Infrastructure › How Infrastructure Needs Change at Scale
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Challenges of Scaling Data Infrastructure › Case Study: Atlassian Engineering Team

