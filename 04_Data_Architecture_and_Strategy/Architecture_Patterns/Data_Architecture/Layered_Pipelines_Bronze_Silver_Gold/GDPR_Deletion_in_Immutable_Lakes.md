---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/GDPR_Deletion_in_Immutable_Lakes"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: []
---

# GDPR Deletion in Immutable Lakes

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Explain what a bronze-layer 'append-only raw' guarantee buys you when you later discover a bug in your silver transformation.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10738_Explain_what_a_bronze-layer_append-only_raw_guarantee_buys_y.md>)
- [Deep] [A GDPR erasure request means a user's rows must vanish, but they're baked into your training set. How does this affect your data architecture and model lifecycle?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10672_A_GDPR_erasure_request_means_a_user_s_rows_must_vanish_but_t.md>)
- [Deep] [Your lake ingestion is idempotent on paper but you keep seeing duplicate training rows after retries. Walk me through where the guarantee actually breaks.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10705_Your_lake_ingestion_is_idempotent_on_paper_but_you_keep_seei.md>)
- [Deep] [Explain why an append-only event lake conflicts with a GDPR/DPDP deletion request, and describe a practical approach your team could implement to actually delete a user.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11076_Explain_why_an_append-only_event_lake_conflicts_with_a_GDPR_.md>)
- [Deep] [How do you architect an append-only immutable event lake so that a bug producing corrupt events can be corrected without violating the immutability contract downstream consumers rely on? Contrast compaction-with-tombstones, correction-events, and versioned rewrites and their consumer-visible semantics.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10953_How_do_you_architect_an_append-only_immutable_event_lake_so_.md>)
- [Deep] [How do you architect an evidence-generating audit trail so proving 'who accessed which sensitive dataset when, and under what policy' is a query over immutable access logs correlated with the catalog? Where does relying on per-engine native audit logs create gaps a determined auditor will find?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11040_How_do_you_architect_an_evidence-generating_audit_trail_so_p.md>)
- [Expert] [How do you architect PII handling across a lake and warehouse so that right-to-erasure (GDPR/DPDP) is actually achievable when data has been copied into hundreds of derived datasets?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10556_How_do_you_architect_PII_handling_across_a_lake_and_warehous.md>)
- [Expert] [Your immutable append-only lake conflicts with GDPR deletion. Reconcile these two hard requirements architecturally — what patterns (crypto-shredding, tombstones, tokenization) do you use and what are their operational costs?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10557_Your_immutable_append-only_lake_conflicts_with_GDPR_deletion.md>)

## Related concepts

- [GDPR Deletion in Feature Pipelines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines.md>) *(Pipeline Orchestration)*
- [Erasure and Model Memorization](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Erasure_and_Model_Memorization.md>) *(Privacy and PII)*
- [Access Governance for LLM Pipelines](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*

