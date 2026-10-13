---
id: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms"
tools: ["Python"]
---

# External Memory Deduplication

**L4 Advanced** · [↑ Data Structures and Algorithms](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [Explain how you'd deduplicate a large list of training samples, and what data structure gives you an average O(1) membership check.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15434_Explain_how_you_d_deduplicate_a_large_list_of_training_sampl.md>)
- [Intermediate] [How would you handle a class-imbalanced dataset by sampling, and what data structure lets you pick from each class in O(1)?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15477_How_would_you_handle_a_class-imbalanced_dataset_by_sampling_.md>)
- [Intermediate] [A batch dedup of 800M training records using a Python set OOMs on the training box; how would you re-architect it with external sorting or a Bloom filter, and what false-positive rate is acceptable for training data?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15529_A_batch_dedup_of_800M_training_records_using_a_Python_set_OO.md>)
- [Intermediate] [You compute a batched softmax over a 2D logits matrix and a naive implementation overflows; how does the log-sum-exp trick keep it numerically stable and why does axis-wise reduction keep it O(n)?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15604_You_compute_a_batched_softmax_over_a_2D_logits_matrix_and_a_.md>)
- [Deep] [How would you compute quantiles for feature binning on a very large column without sorting all of it, and what approximate structure helps?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15451_How_would_you_compute_quantiles_for_feature_binning_on_a_ver.md>)
- [Deep] [A batch dedupe using sort-then-scan works but is I/O bound on huge inputs; when would you switch to hash-based partitioning, and how do you handle partitions that don't fit in memory?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15291_A_batch_dedupe_using_sort-then-scan_works_but_is_I_O_bound_o.md>)
- [Deep] [Your training data pipeline must reservoir-sample k rows from an unbounded stream to build a balanced validation set; prove why the algorithm gives uniform probability and how you extend it to weighted sampling.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15560_Your_training_data_pipeline_must_reservoir-sample_k_rows_fro.md>)
- [Deep] [You need percentile-based feature binning over a huge column without sorting it all; how do approximate quantile sketches (GK or DDSketch) work and what error guarantee do they give for equal-frequency bins?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15570_You_need_percentile-based_feature_binning_over_a_huge_column.md>)

## Related concepts

- [Clickstream Counting Pipeline](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*
- [Idempotent Consumer Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>) *(Low Level Design)*
- [Exact vs Approximate Counting](<../Complexity_Analysis/Exact_vs_Approximate_Counting.md>) *(Complexity Analysis)*

## References

- *Comprehensive Data Structures in C* — Suresh Kumar Srivastava — Introduction

