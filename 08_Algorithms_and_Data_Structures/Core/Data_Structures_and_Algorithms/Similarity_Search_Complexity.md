---
id: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Similarity_Search_Complexity"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms"
tools: []
---

# Similarity Search Complexity

**L4 Advanced** · [↑ Data Structures and Algorithms](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [Why is computing pairwise distances between all points in a dataset O(n^2), and how does that limit naive k-nearest-neighbors at scale?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15440_Why_is_computing_pairwise_distances_between_all_points_in_a_.md>)
- [Basic] [How would you find the single most similar embedding to a query using argmax over a similarity array, and why is that cheaper than sorting the whole array?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15681_How_would_you_find_the_single_most_similar_embedding_to_a_qu.md>)
- [Intermediate] [Explain why brute-force nearest-neighbor search over one million embeddings is O(n*d) per query and why that motivates approximate vector indexes.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15629_Explain_why_brute-force_nearest-neighbor_search_over_one_mil.md>)
- [Intermediate] [How would you compute the average embedding of a set of chunk vectors to get a document-level vector, and what is the time and space complexity?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15666_How_would_you_compute_the_average_embedding_of_a_set_of_chun.md>)
- [Intermediate] [You compute cosine similarity between one query embedding and a million item embeddings on CPU and it's too slow; how does batching into a single matrix multiply change the complexity constant, and where do BLAS libraries help?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15558_You_compute_cosine_similarity_between_one_query_embedding_an.md>)
- [Deep] [How would you find the k centroids furthest apart for k-means++ initialization, and what data structure tracks distances?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15514_How_would_you_find_the_k_centroids_furthest_apart_for_k-mean.md>)
- [Deep] [A payment reconciliation job must match millions of ledger entries to bank statements with fuzzy amount/time keys; how do you structure the matching to avoid O(n*m) and still catch near-matches?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15343_A_payment_reconciliation_job_must_match_millions_of_ledger_e.md>)
- [Deep] [You must find near-duplicate prompts in a logging pipeline to cluster them for eval; why is all-pairs cosine O(n^2) and how does an ANN index or LSH bring clustering down to a tractable cost?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15754_You_must_find_near-duplicate_prompts_in_a_logging_pipeline_t.md>)

## Related concepts

- [Exact vs Approximate Counting](<../Complexity_Analysis/Exact_vs_Approximate_Counting.md>) *(Complexity Analysis)*
- [Vector Similarity in SQL](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL.md>) *(SQL Fundamentals)*
- [Chunk Size and Retrieval Quality](<../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Chunk_Size_and_Retrieval_Quality.md>) *(Retrieval and RAG)*

## References

- *Essential GraphRAG* — Tomaž Bratanič, Oskar Hane — Advanced vector retrieval strategies 31
- *Essential GraphRAG* — Tomaž Bratanič, Oskar Hane — Advanced vector retrieval strategies 33

