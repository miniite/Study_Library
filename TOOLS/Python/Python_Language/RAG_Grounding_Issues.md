---
id: "TOOLS/Python/Python_Language/RAG_Grounding_Issues"
level: 4
type: "concept"
topic: "TOOLS/Python/Python_Language"
tools: ["Python"]
---

# RAG Grounding Issues

**L4 Advanced** · [↑ Python Language](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../_index.md>)

## Practice questions

- [Basic] [What is Retrieval-Augmented Generation (RAG) in one sentence, and what are the two main Python steps: retrieve then generate?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37384_What_is_Retrieval-Augmented_Generation_RAG_in_one_sentence_a.md>)
- [Intermediate] [How would you query a vector database from Python to retrieve the top-k most similar chunks for a user question?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37382_How_would_you_query_a_vector_database_from_Python_to_retriev.md>)
- [Intermediate] [What is the difference between keyword (lexical) search and semantic (embedding) search, and when would you combine them in a Python retrieval step?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37441_What_is_the_difference_between_keyword_lexical_search_and_se.md>)
- [Intermediate] [A stakeholder complains your RAG bot cites the wrong document. How would you instrument the Python pipeline to attach and verify source attribution so a citation always maps to the chunk that produced the claim?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37469_A_stakeholder_complains_your_RAG_bot_cites_the_wrong_documen.md>)
- [Deep] [How would you structure a Python retrieval layer so you can swap between dense, sparse (BM25), and hybrid search behind one interface, and how do you fuse the two score distributions?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37466_How_would_you_structure_a_Python_retrieval_layer_so_you_can_.md>)
- [Deep] [Your RAG retrieves relevant chunks but the LLM ignores them and answers from parametric memory. How would you diagnose this in Python and change prompting or context ordering to force grounding?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37501_Your_RAG_retrieves_relevant_chunks_but_the_LLM_ignores_them_.md>)
- [Deep] [Your embedding-based dedup misses near-duplicate documents that hurt retrieval diversity. How would you implement MMR or a diversity-aware re-ranking in Python to reduce redundant chunks in the context?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37512_Your_embedding-based_dedup_misses_near-duplicate_documents_t.md>)
- [Deep] [Your Python RAG answers are correct but cite the wrong source chunk, eroding trust. How would you architect citation/attribution tracking through embedding, retrieval and generation so the cited span is provably the one used?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37596_Your_Python_RAG_answers_are_correct_but_cite_the_wrong_sourc.md>)

## Related concepts

- [Re-ranking](<../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Re-ranking.md>) *(Retrieval and RAG)*
- [Inverted Indexes](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>) *(Data Structures and Algorithms)*
- [Chunk Size and Retrieval Quality](<../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Chunk_Size_and_Retrieval_Quality.md>) *(Retrieval and RAG)*

## References

- *Essential GraphRAG* — Tomaž Bratanič, Oskar Hane — Knowledge Graph-Enhanced RAG

