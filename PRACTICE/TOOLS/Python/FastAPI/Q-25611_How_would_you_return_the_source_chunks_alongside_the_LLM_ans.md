---
type: "practice"
difficulty: "Intermediate"
---

# How would you return the source chunks alongside the LLM answer so a client can see the citations from your FastAPI RAG endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we'll need to ensure that our FastAPI application not only fetches the information needed by the Large Language Model (LLM) but also returns the relevant source chunks with the generated answer. This involves integrating a Retrieval-Augmented Generation (RAG) system where the LLM can be prompted using source documents or knowledge bases, and then both the response and the sources used are returned to the client.

The key idea here is that we need to structure our API response such that it includes both the generated answer and the corresponding source chunks. This will help users understand which parts of their input were used by the LLM in generating the output, enhancing transparency and traceability.

**Comparison:**

```
Query	Answer with Sources
What are the benefits of using FastAPI?	{ answer: "FastAPI is a modern, fast (high-performance) web framework for building APIs with Python 3.7+ based on standard Python type hints.", sources: [ chunk1, chunk2 ] }
```

**Code:**

```

from fastapi import FastAPI, Depends, HTTPException from pydantic import BaseModel import requests

app = FastAPI()

Mock class to simulate the retrieval of source chunks

class SourceChunk: def init(self, text, document_id): self.text = text self.document_id = document_id

Mock function to retrieve context from RAG system

def get_context(query: str) -> list[SourceChunk]: # Here we would actually fetch data from a database or external service return [SourceChunk("This is chunk 1", "doc_1"), SourceChunk("This is chunk 2", "doc_2")]

Pydantic model for the API response

class AnswerWithSources(BaseModel): answer: str sources: list[SourceChunk]

@app.post("/answer/") async def get_answer(query: str) -> AnswerWithSources: # Fetch context from RAG system context = get_context(query)

CODE
Copy
# Simulate LLM processing (for simplicity, we just return the query as answer with context)
response = query  # Placeholder for actual model output

# Prepare and return the answer along with sources
return AnswerWithSources(answer=response, sources=context)
CODE
Copy

```python
```

**Example input / output:**

```text
# Sample input and output
query = "What are the benefits of using FastAPI?"
response = get_answer(query)
print(response)

Output:
{
    "answer": "What are the benefits of using FastAPI?",
    "sources": [
        {
            "text": "This is chunk 1",
            "document_id": "doc_1"
        },
        {
            "text": "This is chunk 2",
            "document_id": "doc_2"
        }
    ]
}
```

**Data:**

```json
[
    {
        "text": "This is chunk 1",
        "document_id": "doc_1"
    },
    {
        "text": "This is chunk 2",
        "document_id": "doc_2"
    }
]
```

**Flow:**

1. Client Request
2. GET /answer/
3. RAG System Context Retrieval
4. LLM Response Generation
5. Answer & SourcesConstruction
6. API Response

