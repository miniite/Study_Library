---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/Partial_Failure_in_Fan-out_APIs"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Python"]
---

# Partial Failure in Fan-out APIs

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Intermediate] [In a RAG system, what REST calls happen between the user asking a question and the LLM producing an answer? Walk through the sequence.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38763_In_a_RAG_system_what_REST_calls_happen_between_the_user_aski.md>)
- [Intermediate] [Why is document ingestion for RAG usually handled as a separate REST endpoint from the actual question-answering endpoint?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38778_Why_is_document_ingestion_for_RAG_usually_handled_as_a_separ.md>)
- [Intermediate] [What is the 202 Accepted status code, and how could a RAG document-processing endpoint use it while embeddings are being generated in the background?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38780_What_is_the_202_Accepted_status_code_and_how_could_a_RAG_doc.md>)
- [Intermediate] [Why should a RAG API scope every vector search to the logged-in user's own documents, and what security problem happens if you forget?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38801_Why_should_a_RAG_API_scope_every_vector_search_to_the_logged.md>)
- [Intermediate] [What does over-fetching mean when your RAG endpoint returns full document chunks the client does not need, and how would you trim the response?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38840_What_does_over-fetching_mean_when_your_RAG_endpoint_returns_.md>)
- [Intermediate] [Why is returning source citations in a RAG API response useful, and how would you structure them in the JSON?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38842_Why_is_returning_source_citations_in_a_RAG_API_response_usef.md>)
- [Deep] [Two clients send the same expensive RAG query within milliseconds and you pay for two full pipelines. How would you add in-flight request coalescing on the server so duplicate concurrent queries share one generation?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38851_Two_clients_send_the_same_expensive_RAG_query_within_millise.md>)
- [Deep] [Your ingestion REST endpoint accepts a 200-page PDF for RAG. Chunking and embedding take minutes and the client can't wait. Design the async ingestion contract including progress reporting and how the client knows which chunks failed.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38871_Your_ingestion_REST_endpoint_accepts_a_200-page_PDF_for_RAG_.md>)

## Related concepts

- [RAG Endpoints in FastAPI](<../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>) *(FastAPI)*
- [RAG Grounding Issues](<../../../TOOLS/Python/Python_Language/RAG_Grounding_Issues.md>) *(Python Language)*
- [RAG Corpus Governance](<../../../05_Data_Governance_and_Security/Governance/Data_Strategy/RAG_Corpus_Governance.md>) *(Data Strategy)*

