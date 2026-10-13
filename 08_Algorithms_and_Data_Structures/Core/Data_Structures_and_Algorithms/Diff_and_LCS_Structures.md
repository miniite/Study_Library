---
id: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Diff_and_LCS_Structures"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms"
tools: []
---

# Diff and LCS Structures

**L4 Advanced** · [↑ Data Structures and Algorithms](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [When building a RAG pipeline you split documents into chunks; what data structure would you use to keep each chunk alongside its source metadata and why?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15630_When_building_a_RAG_pipeline_you_split_documents_into_chunks.md>)
- [Basic] [What data structure would you use to remember which prompts have already been sent so you can skip re-sending identical ones in a batch, and why?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15695_What_data_structure_would_you_use_to_remember_which_prompts_.md>)
- [Intermediate] [Your RAG citations must map each returned chunk back to its source document and offset in O(1) after a vector search returns IDs; what structure holds the chunk-to-source mapping and how do you keep it in sync on re-ingest?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15760_Your_RAG_citations_must_map_each_returned_chunk_back_to_its_.md>)
- [Deep] [A prod service serializes and diffs large config trees on every reload and CPU spikes; how would you use a Merkle-tree hash to skip unchanged subtrees and where does the hashing cost pay off?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15340_A_prod_service_serializes_and_diffs_large_config_trees_on_ev.md>)
- [Deep] [A model config store must serve 'the version of these features as of timestamp T' for reproducible training; compare full snapshots, deltas, and a persistent (immutable) tree for storage and read cost.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15582_A_model_config_store_must_serve_the_version_of_these_feature.md>)
- [Deep] [A hybrid retriever must intersect candidate chunk IDs from a metadata filter with those from vector search before ranking; compare sorted-merge, hash-set probing, and roaring bitmaps for a filter that matches millions of chunks.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15753_A_hybrid_retriever_must_intersect_candidate_chunk_IDs_from_a.md>)
- [Expert] [How would you design a data structure to support 'get the version of this document as of timestamp T' for a document that changes frequently, keeping storage and read cost reasonable?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15282_How_would_you_design_a_data_structure_to_support_get_the_ver.md>)
- [Expert] [A model retraining triggers on data changes and you must diff two large feature-config trees efficiently; how would a Merkle-tree hash let you skip unchanged subtrees, and where does the hashing cost pay off?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15555_A_model_retraining_triggers_on_data_changes_and_you_must_dif.md>)

## Related concepts

- [Clickstream Counting Pipeline](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*
- [Exact vs Approximate Counting](<../Complexity_Analysis/Exact_vs_Approximate_Counting.md>) *(Complexity Analysis)*
- [Distributed Cache Design](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

