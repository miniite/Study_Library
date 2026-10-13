---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Hub-and-spoke_Cross-cloud_Networking"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data"
tools: ["AWS", "Azure", "Google_Cloud"]
---

# Hub-and-spoke Cross-cloud Networking

**L5 Expert** · [↑ Multi Cloud Data](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Azure](<../../../TOOLS/Azure/_index.md>)
- [Google Cloud](<../../../TOOLS/Google_Cloud/_index.md>)

## Practice questions

- [Deep] [Your CFO wants a single pane of glass for spend across AWS Cost Explorer / GCP Billing / Azure Cost Management — how do you build cross-cloud FinOps that attributes cost down to a data product?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32899_Your_CFO_wants_a_single_pane_of_glass_for_spend_across_AWS_C.md>)
- [Deep] [Design a chargeback model that fairly bills domain teams for cross-cloud data-transfer costs they cause — how do you attribute egress to the consumer who triggered it?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32945_Design_a_chargeback_model_that_fairly_bills_domain_teams_for.md>)
- [Deep] [How do you approach the total-cost-of-ownership model for multi-cloud data — what hidden costs (talent, tooling duplication, integration, egress) do teams consistently underestimate?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32976_How_do_you_approach_the_total-cost-of-ownership_model_for_mu.md>)
- [Deep] [What's your strategy for benchmarking price/performance of the same analytical workload on two clouds fairly — how do you avoid a benchmark that flatters whichever vendor sponsored it?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32996_What_s_your_strategy_for_benchmarking_price_performance_of_t.md>)
- [Expert] [Explain your approach to cross-cloud encryption key management — do you centralize on one cloud's KMS, run an external KMS, or federate, and what does each choice do to your blast radius?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32918_Explain_your_approach_to_cross-cloud_encryption_key_manageme.md>)
- [Expert] [Your leadership wants to avoid lock-in but your fastest path uses Snowflake's proprietary features — how do you frame the lock-in-vs-velocity tradeoff as a reversible-vs-irreversible-decisions problem?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32932_Your_leadership_wants_to_avoid_lock-in_but_your_fastest_path.md>)
- [Expert] [How do you approach vendor risk when a critical managed service (say Confluent Cloud) is only excellent on one cloud — do you accept the coupling or self-host for parity across clouds?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32953_How_do_you_approach_vendor_risk_when_a_critical_managed_serv.md>)
- [Expert] [How do you decide the granularity of replication — whole datasets, changed partitions, or CDC row-level — for a cross-cloud analytics copy, and what does each cost you to keep consistent?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32966_How_do_you_decide_the_granularity_of_replication_whole_datas.md>)

## Related concepts

- [Multi-cloud Strategy](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*
- [Commitments and Spot Capacity](<../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Commitments_and_Spot_Capacity.md>) *(FinOps)*
- [Open vs Proprietary Formats](<../../Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>) *(Data Platform Strategy)*

## References

- *System Design Big Archive (2023)* — ByteByteGo — Cloud Native Anti Patterns
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Trade-offs between public cloud, on-premises, and multi-cloud

