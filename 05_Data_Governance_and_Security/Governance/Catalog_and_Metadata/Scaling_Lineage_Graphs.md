---
id: "05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Scaling_Lineage_Graphs"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Governance/Catalog_and_Metadata"
tools: []
---

# Scaling Lineage Graphs

**L5 Expert** · [↑ Catalog and Metadata](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [Why is push-based lineage capture (agents in the compute engines) fundamentally different in reliability and coverage from pull/parse-based lineage, and when would you accept the weaker one?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11142_Why_is_push-based_lineage_capture_agents_in_the_compute_engi.md>)
- [Deep] [Probe: an interviewer says 'lineage is 90% coverage — good enough, ship it.' Where is that dangerous? Which use cases require near-100% lineage completeness and how do you communicate coverage confidence to consumers?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11172_Probe_an_interviewer_says_lineage_is_90_coverage_good_enough.md>)
- [Deep] [Probe: your popularity ranking creates a rich-get-richer loop — popular tables get more visible, get used more, get more popular — while a better new table stays invisible. How do you design ranking to avoid entrenching legacy datasets?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11235_Probe_your_popularity_ranking_creates_a_rich-get-richer_loop.md>)
- [Expert] [Design how tags/classifications propagate through lineage automatically — if an upstream column is tagged PII, downstream derived columns should inherit it. How do you prevent over-propagation (everything becomes PII) and under-propagation?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11158_Design_how_tags_classifications_propagate_through_lineage_au.md>)
- [Expert] [Design your approach for cross-domain lineage stitching in a mesh where each domain emits its own lineage independently — how do you join partial lineage graphs at domain boundaries into one coherent end-to-end view?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11174_Design_your_approach_for_cross-domain_lineage_stitching_in_a.md>)
- [Expert] [Why might a graph database be the wrong choice for your lineage/metadata store despite lineage being 'a graph'? What alternatives (relational + adjacency, document, columnar, hybrid) would you weigh?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11184_Why_might_a_graph_database_be_the_wrong_choice_for_your_line.md>)
- [Expert] [You need column-level lineage across SQL, Spark, dbt, and hand-written Python jobs. Where does automated lineage extraction break down, and how do you close the gaps without asking every team to annotate manually?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11193_You_need_column-level_lineage_across_SQL_Spark_dbt_and_hand-.md>)
- [Expert] [Probe: two teams cataloged 'the same' customer table under different URNs, so lineage forks and search returns both. How do you design entity dedup/merge without corrupting downstream references?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11212_Probe_two_teams_cataloged_the_same_customer_table_under_diff.md>)

## Related concepts

- [Lineage for Dynamic SQL](<../Lineage/Lineage_for_Dynamic_SQL.md>) *(Lineage)*
- [Distinguishing Drift from Pipeline Breaks](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>) *(Data Architecture)*
- [Event Schema Evolution](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>) *(Distributed Systems Core)*

## References

- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Building End-to-End Lineage

