---
type: "practice"
difficulty: "Basic"
---

# Why can't you just feed an entire long document straight into an embedding model?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Structure Aware Chunking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Structure_Aware_Chunking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with long documents in natural language processing (NLP) tasks, it's not practical or effective to feed the entire document directly into an embedding model due to several fundamental limitations and challenges. This approach can lead to inefficiencies, high computational costs, and potential issues with context and meaning preservation.

#### Triage

The question is probing why feeding a long document as-is into an embedding model (like BERT or any other transformer-based model) isn't feasible without first breaking the document down into smaller chunks. The key assumptions here are that the embeddings need to capture meaningful representations of the text, which can be challenging for very long inputs.

#### First-Principles Solution
Embedding Model Limitations: Most embedding models like BERT and others are designed to handle fixed-length input sequences. A long document exceeds this limitation.
Contextual Preservation: Long documents may contain complex contextual relationships that could get lost when the model processes large inputs in one go.
Computational Efficiency: Processing a single long document can be computationally expensive, leading to increased time and resource usage.

#### Solution

To address these issues, we use a technique called "chunking" followed by "retrieval strategies." Here’s how it works:

Chunking:

Break down the long document into smaller, manageable chunks (typically sentences or paragraphs).
Ensure that each chunk captures some meaningful segment of the text without losing context.

Embedding Each Chunk:

Feed each chunk individually to the embedding model.
This allows the model to process and understand the local context within each chunk effectively.

Retrieval Strategies:

After obtaining embeddings for all chunks, a retrieval strategy is applied to aggregate or compare these embeddings based on their relevance.
Techniques like attention mechanisms can help in understanding how different parts of the document are interrelated.

#### Closing Summary
and Common Mistakes/Follow-Ups

The common mistake is assuming that feeding long documents directly into an embedding model will yield accurate results, especially for tasks requiring deep contextual understanding. Always chunking the input not only helps with computational efficiency but also ensures better preservation of meaning within each segment.

Some follow-up questions to consider:

How does the choice of chunk size affect the overall performance?
What are some retrieval strategies that can be used after embedding chunks?

**Comparison:**

```
Chunk	Embedding
1. "This is a very long document that needs to be"	[0.12, 0.23, ...]
2. "processed efficiently. Splitting it into smaller"	[-0.15, -0.08, ...]
3. "chunks helps with both computational efficiency"	[0.45, 0.67, ...]
4. "and context preservation."	[-0.29, 0.31, ...]
```

**Code:**

```python
# Example code for splitting a long document into chunks and processing them
def split_document_into_chunks(document, max_chunk_length=128):
    chunks = [document[i:i+max_chunk_length] for i in range(0, len(document), max_chunk_length)]
    return chunks

chunks = split_document_into_chunks("This is a very long document that needs to be processed efficiently. Splitting it into smaller chunks helps with both computational efficiency and context preservation.")
```

**Example input / output:**

```text
Input Document:
"This is a very long document that needs to be processed efficiently. Splitting it into smaller chunks helps with both computational efficiency and context preservation."

Processed Chunks:
1. "This is a very long document that needs to be"
2. "processed efficiently. Splitting it into smaller"
3. "chunks helps with both computational efficiency"
4. "and context preservation."
```

**Data:**

```json
{
  "document": "This is a very long document that needs to be processed efficiently. Splitting it into smaller chunks helps with both computational efficiency and context preservation.",
  "chunks": [
    "This is a very long document that needs to be",
    "processed efficiently. Splitting it into smaller",
    "chunks helps with both computational efficiency",
    "and context preservation."
  ]
}
```

**Flow:**

1. Yes
2. Input Long Document
3. Split into Chunks?
4. Process Each Chunk
5. Aggregate Embeddings UsingRetrieval Strategy
6. Output ContextualRepresentation

