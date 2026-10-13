---
id: "04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management"
tools: []
---

# Golden Records and Survivorship

**L5 Expert** · [↑ Master Data Management](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [When you say 'golden record', what is the actual data model and lineage behind it — do you materialize one physical golden record, keep source-attributed survivorship, or expose a virtual golden view, and what drove that choice?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-31991_When_you_say_golden_record_what_is_the_actual_data_model_and.md>)
- [Deep] [Explain your approach to survivorship conflict resolution when two authoritative sources disagree on a legally-significant attribute (e.g., tax ID). Do you pick, flag, or escalate — and who is accountable?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32024_Explain_your_approach_to_survivorship_conflict_resolution_wh.md>)
- [Deep] [How do you handle late-arriving and slowly-corrected source data — a source fixes a typo six months later — such that the golden record heals without erasing the correct decision history?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32037_How_do_you_handle_late-arriving_and_slowly-corrected_source_.md>)
- [Deep] [How do you decide between attribute-level trust scores propagated to consumers vs a single binary 'validated' flag on the golden record — what do downstream consumers actually need to act safely?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32080_How_do_you_decide_between_attribute-level_trust_scores_propa.md>)
- [Expert] [What is your survivorship and merge/unmerge strategy, and how do you make unmerge reversible and auditable years later when a false merge is discovered?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-31993_What_is_your_survivorship_and_merge_unmerge_strategy_and_how.md>)
- [Expert] [What's your strategy for blocking / candidate generation so you don't do O(n^2) comparisons at hundreds of millions of records — and how do you validate you're not losing true matches in the blocking step?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32017_What_s_your_strategy_for_blocking_candidate_generation_so_yo.md>)
- [Expert] [How do you keep master data trustworthy as ground truth for a RAG/agent system — what happens to answers when the golden record is wrong, and how do you gate LLM access to master data?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32023_How_do_you_keep_master_data_trustworthy_as_ground_truth_for_.md>)
- [Expert] [How do you validate the quality of your entity resolution without ground-truth labels — what evaluation strategy (blocking recall, pairwise precision, cluster metrics, human sampling) do you trust at scale?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32045_How_do_you_validate_the_quality_of_your_entity_resolution_wi.md>)

## Related concepts

- [Data Vault](<../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Quality SLAs and Rule Testing](<../../../02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>) *(Quality Frameworks and SLAs)*
- [Event Schema Evolution](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>) *(Distributed Systems Core)*

