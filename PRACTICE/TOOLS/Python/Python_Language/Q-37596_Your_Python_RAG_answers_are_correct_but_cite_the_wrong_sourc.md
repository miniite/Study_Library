---
type: "practice"
difficulty: "Deep"
---

# Your Python RAG answers are correct but cite the wrong source chunk, eroding trust. How would you architect citation/attribution tracking through embedding, retrieval and generation so the cited span is provably the one used?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ RAG Grounding Issues](<../../../../TOOLS/Python/Python_Language/RAG_Grounding_Issues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of correctly attributing the source chunks in a Retrieval-Augmented Generation (RAG) system, we need to ensure that the citation tracking process is robust and verifiable. This involves embedding the source documents, retrieving relevant embeddings, and generating responses while maintaining clear attribution. Here’s how we can architect this solution from first principles.

Firstly, let's clarify what the problem is: The current RAG system generates answers correctly but often cites wrong source chunks, which erodes user trust. To solve this, we need to ensure that the cited span in the generated response is provably correct by tracking and matching it back to its original source document.

#### Triage

The core of the question involves embedding the source documents, retrieving relevant embeddings during retrieval, and generating responses while maintaining clear attribution. We'll assume we have a collection of source documents, an embedding model, and a retrieval system. The goal is to ensure that any generated response cites the correct original span from the document.

Step-by-Step Solution
Document Embedding: Embed each source document into a high-dimensional space where similar content has close embeddings.
Retrieval System: Use the embedding model to retrieve relevant chunks of text when a query is made.
Span Tracking and Generation: During generation, maintain a clear mapping between generated output spans and their original source document spans. Ensure that any cited span can be traced back to its exact location in the original document.
1. Document Embedding

We'll use a pre-trained embedding model like BERT or SentenceTransformer to embed each document into a high-dimensional vector space. This will allow us to measure similarity between documents and retrieve relevant chunks efficiently.

PYTHON
from sentence_transformers import SentenceTransformer

embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
documents = ["Document 1 content", "Document 2 content", ...]
embeddings = [embedding_model.encode(doc) for doc in documents]
2. Retrieval System

For retrieval, we'll use a high-performance retrieval system that can efficiently query the embedding space and return relevant chunks of text. This could be implemented using an approximate nearest neighbor (ANN) search algorithm like Annoy or Faiss.

PYTHON
import faiss

index = faiss.IndexFlatL2(len(embeddings[0]))
for doc_emb in embeddings:
    index.add([doc_emb])
3. Span Tracking and Generation

When generating a response, we'll maintain a mapping between the generated text spans and their original source document spans. This can be achieved by tracking the retrieved chunks during the retrieval phase and ensuring that any citation is correctly attributed.

PYTHON
def generate_response(query):
    # Retrieve relevant chunks
    query_embedding = embedding_model.encode(query)
    distances, indices = index.search([query_embedding], k=3)

    relevant_chunks = [documents[i] for i in indices[0]]

    # Generate response with attribution
    generated_response, attributed_spans = generate_response_with_attribution(relevant_chunks)
    return generated_response, attributed_spans

def generate_response_with_attribution(chunks):
    # Implement the generation logic and attribution tracking here
    ...
    return response, attributed_spans

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we've outlined a robust system for attributing source chunks in RAG by embedding documents, retrieving relevant chunks efficiently, and maintaining clear attribution during response generation. Key mistakes to avoid include not properly embedding documents, using an inefficient retrieval method, or failing to track the original spans accurately.

Common follow-ups might involve discussing specific embedding techniques, tuning the retrieval algorithm, handling large-scale document collections, and ensuring real-time performance.

**Comparison:**

```
Query	Generated Response	Attributed Span
What is the latest research on AI ethics?	The latest research on AI ethics includes ... (cited from Document 2)	[Document 2, p.5, line 10]
```

**Code:**

```python
from sentence_transformers import SentenceTransformer

embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
documents = ["Document 1 content", "Document 2 content", ...]
embeddings = [embedding_model.encode(doc) for doc in documents]

import faiss

index = faiss.IndexFlatL2(len(embeddings[0]))
for doc_emb in embeddings:
    index.add([doc_emb])

def generate_response(query):
    query_embedding = embedding_model.encode(query)
    distances, indices = index.search([query_embedding], k=3)

    relevant_chunks = [documents[i] for i in indices[0]]

    generated_response, attributed_spans = generate_response_with_attribution(relevant_chunks)
    return generated_response, attributed_spans

def generate_response_with_attribution(chunks):
    # Implement the generation logic and attribution tracking here
    ...
    return response, attributed_spans
```

**Example input / output:**

```text
query = "What is the latest research on AI ethics?"
response, attributed_spans = generate_response(query)
print("Generated Response:", response)
print("Attributed Spans:", attributed_spans)
```

**Data:**

```python
documents = ["Document 1 content", "Document 2 content", ...]
embeddings = [embedding_model.encode(doc) for doc in documents]
```

**Flow:**

1. Query
2. Retrieve relevant chunks
3. Generate response withattribution
4. Response
5. Attributed Spans

