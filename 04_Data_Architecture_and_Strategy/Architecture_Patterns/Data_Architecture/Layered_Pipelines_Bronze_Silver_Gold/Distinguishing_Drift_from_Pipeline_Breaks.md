---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Distinguishing_Drift_from_Pipeline_Breaks"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS"]
---

# Distinguishing Drift from Pipeline Breaks

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Explain what lineage is and give a concrete example where lineage saved you when a downstream model started misbehaving.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10713_Explain_what_lineage_is_and_give_a_concrete_example_where_li.md>)
- [Intermediate] [Your company wants 'a data catalog.' As the engineer, what would actually go in it, how would you keep it current, and what breaks if you rely on people to update it manually?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11056_Your_company_wants_a_data_catalog_As_the_engineer_what_would.md>)
- [Deep] [You're asked to consume a 'customer' data product owned by another team for your churn model. What do you check about its data contract before you build on it?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10652_You_re_asked_to_consume_a_customer_data_product_owned_by_ano.md>)
- [Deep] [Explain the difference between data quality checks and data observability. If you could only instrument three signals on your gold tables first, which and why?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11070_Explain_the_difference_between_data_quality_checks_and_data_.md>)
- [Deep] [Architect discoverability and trust signals for ML data products so a model team can self-serve the right feature without pinging producers. What metadata, certification, and usage-signal loop makes a feature mesh actually usable at 200k assets?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10794_Architect_discoverability_and_trust_signals_for_ML_data_prod.md>)
- [Expert] [Define a data contract precisely. What does it cover beyond schema, how is it enforced in CI/CD, and what happens at runtime when a producer breaks it? Who is paged?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10550_Define_a_data_contract_precisely_What_does_it_cover_beyond_s.md>)
- [Expert] [Explain how you'd resolve a cross-team dispute where the platform team wants to enforce a canonical event schema and product teams want freedom. What's your decision framework and how do you avoid it recurring every quarter?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10642_Explain_how_you_d_resolve_a_cross-team_dispute_where_the_pla.md>)
- [Expert] [How would you architect the platform to distinguish real concept/data drift from an upstream schema or pipeline change masquerading as drift? What lineage and monitoring integration lets you triage this org-wide instead of per-model?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10800_How_would_you_architect_the_platform_to_distinguish_real_con.md>)

## Related concepts

- [Governing Real-time Features](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>) *(Stewardship)*
- [Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>) *(Distributed Systems Core)*
- [Governing Synthetic Training Data](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Synthetic_Training_Data.md>) *(Data Strategy)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Preventing the Data Lake Abyss How to Ensure That Your Data Remains Valid Over the Years
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Summary
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Introducing Database Reliability Engineering
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — A Data Architecture Sampler

