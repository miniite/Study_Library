---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/Vector_Search_APIs"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Python"]
---

# Vector Search APIs

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is the difference between an embeddings REST endpoint and a chat completion REST endpoint in terms of what you send and what you get back?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38757_What_is_the_difference_between_an_embeddings_REST_endpoint_a.md>)
- [Basic] [How do vector databases like Pinecone or Qdrant expose their functionality over REST? Name the kinds of operations you would call.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38760_How_do_vector_databases_like_Pinecone_or_Qdrant_expose_their.md>)
- [Intermediate] [Why do embeddings endpoints usually let you send an array of texts in one request instead of one text at a time?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38759_Why_do_embeddings_endpoints_usually_let_you_send_an_array_of.md>)
- [Intermediate] [How would you design a single /ask REST endpoint for a RAG chatbot that internally calls an embedding API, a vector DB, and an LLM?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38764_How_would_you_design_a_single_ask_REST_endpoint_for_a_RAG_ch.md>)
- [Intermediate] [How would you design a REST query to a vector database that both does similarity search and filters by metadata, for example only documents from a certain user?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38800_How_would_you_design_a_REST_query_to_a_vector_database_that_.md>)
- [Intermediate] [How would you log and trace a single RAG request as it flows through your embedding, vector search, and LLM REST calls?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38829_How_would_you_log_and_trace_a_single_RAG_request_as_it_flows.md>)
- [Intermediate] [How would you design an /embeddings endpoint on your own backend that batches many texts and returns their vectors efficiently?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38839_How_would_you_design_an_embeddings_endpoint_on_your_own_back.md>)
- [Intermediate] [You expose an /embeddings REST endpoint that batches texts, but one oversized text in a batch fails the whole call. How do you design partial-success semantics so valid texts return vectors and only the bad ones are reported?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38905_You_expose_an_embeddings_REST_endpoint_that_batches_texts_bu.md>)

## Related concepts

- [RAG Endpoints in FastAPI](<../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>) *(FastAPI)*
- [Vector Data in the Lakehouse](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Vector_Data_in_the_Lakehouse.md>) *(Data Architecture)*
- [RAG Grounding Issues](<../../../TOOLS/Python/Python_Language/RAG_Grounding_Issues.md>) *(Python Language)*

