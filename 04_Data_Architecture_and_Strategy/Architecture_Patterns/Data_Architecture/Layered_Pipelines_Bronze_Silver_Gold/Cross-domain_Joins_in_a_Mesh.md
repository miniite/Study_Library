---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Cross-domain_Joins_in_a_Mesh"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS"]
---

# Cross-domain Joins in a Mesh

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [You're told the team is 'moving to data mesh.' As a mid-level engineer whose team now owns its data as a product, what actually changes about your day-to-day responsibilities?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11065_You_re_told_the_team_is_moving_to_data_mesh_As_a_mid-level_e.md>)
- [Deep] [A dashboard needs data joined across two domain-owned products in a mesh. Do you replicate their product into your area or federate-query it live? How do you decide?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11093_A_dashboard_needs_data_joined_across_two_domain-owned_produc.md>)
- [Deep] [A domain team is told to own its feature products end-to-end but has no data-engineering skill. How do you resolve the tension between decentralized ML data ownership and uneven team maturity? Define exactly what the paved road provides for free.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10791_A_domain_team_is_told_to_own_its_feature_products_end-to-end.md>)
- [Deep] [How do you architect a 'bring compute to data' model for a mesh where domains keep their data products in their own accounts/regions, and consumers query cross-account? What are the IAM, cost-attribution, and network-egress traps that make this harder than it looks?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10978_How_do_you_architect_a_bring_compute_to_data_model_for_a_mes.md>)
- [Deep] [Explain how you'd architect a chargeback model that's fair when compute, storage, and data-transfer are all shared and gameable. What query-tagging, storage-attribution, and cross-domain-read accounting design makes a domain's bill reflect its true cost and can't be shifted onto a shared pool?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11036_Explain_how_you_d_architect_a_chargeback_model_that_s_fair_w.md>)
- [Expert] [Two domains model 'customer' differently and both publish a 'customer' data product. As the principal, how do you resolve the semantic conflict without imposing a central data model that nobody uses?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10558_Two_domains_model_customer_differently_and_both_publish_a_cu.md>)
- [Expert] [How do you architect cross-domain joins in a mesh where each domain owns its data product? What's the performance and governance cost of federated query vs replicating data products, and when do you choose each?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10586_How_do_you_architect_cross-domain_joins_in_a_mesh_where_each.md>)
- [Expert] [Your mesh is live but adoption stalled — domains treat their data products as an afterthought. Diagnose the incentive/organizational failure and design the mechanisms (metrics, funding, recognition) that fix it.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10614_Your_mesh_is_live_but_adoption_stalled_domains_treat_their_d.md>)

## Related concepts

- [Domain Ownership vs Cross-cutting Data](<../../Data_Mesh/Domain_Ownership_vs_Cross-cutting_Data.md>) *(Data Mesh)*
- [Data Mesh Principles](<../../Data_Mesh/Data_Mesh_Principles.md>) *(Data Mesh)*
- [Federated vs Centralized Data Ownership](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Federated_vs_Centralized_Data_Ownership.md>) *(Distributed Systems Core)*

## References

- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Data Sharing and Collaboration
- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — Architecture and Data Topologies
- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — Architecture and Team Topologies
- *Software Architecture: The Hard Parts* — Neal Ford, Mark Richards, Pramod Sadalage, Zhamak Dehghani — Data Ownership and Distributed Transactions

