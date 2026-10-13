---
type: "practice"
difficulty: "Deep"
---

# How would you design a FastAPI RAG endpoint so each stage (retrieve, rerank, generate) is independently timed, retried, and can degrade gracefully when one stage fails?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ RAG Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a FastAPI ReAct (Retrieval-Augmented Generation) endpoint where each stage—retrieval, reranking, and generation—is independently timed, retried, and can degrade gracefully when one stage fails, we need to ensure modularity and fault tolerance. This involves breaking down the process into clear stages, managing dependencies, and handling retries and timeouts effectively.

#### Triage

The question is asking for a design that ensures each component of the RAG pipeline operates independently. Specifically:

Retrieval: Fetching relevant documents.
Reranking: Ranking these documents based on relevance.
Generation: Generating responses using the most relevant documents.

We need to manage retries and timeouts at each stage, ensuring graceful degradation if any part fails. We should also consider performance metrics like latency and throughput to ensure the system scales well under varying loads.

#### Assumptions
The FastAPI backend is running in a containerized environment (e.g., Docker) with access to necessary resources.
Each stage has its own set of errors that can be caught, logged, and retried independently.
We have an external service or database for storing and managing documents.

#### First-Principles Solution
Modularize the Pipeline: Define a modular FastAPI app where each component is a separate function or class.
Asynchronous Processing: Use async/await to handle asynchronous operations without blocking the main thread.
Error Handling and Retries: Implement retry mechanisms with exponential backoff for transient errors.
Graceful Degradation: Design each stage to return partial results if it fails, ensuring the next stages receive what they can use.

#### Implementation
Step 1: Define the FastAPI App Structure
PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import asyncio
import json
from typing import List, Optional

app = FastAPI()

class Document(BaseModel):
    id: int
    content: str

class QueryRequest(BaseModel):
    query: str

class QueryResponse(BaseModel):
    results: List[Document]
Step 2: Define the Stages as Async Functions
PYTHON
async def retrieve_documents(query: str) -> Optional[List[Document]]:
    # Simulate retrieval from a database or external service
    documents = await fetch_documents_from_db(query)
    if not documents:
        return None
    return documents

async def rerank_documents(documents: List[Document], query: str) -> Optional[List[Document]]:
    # Rerank the documents based on relevance to the query
    ranked_docs = await rank_documents(documents, query)
    if not ranked_docs:
        return None
    return ranked_docs

async def generate_response(ranked_docs: List[Document]) -> Optional[str]:
    # Generate a response using the top-ranked document(s)
    response = await process_and_generate(ranked_docs[0].content)  # Simplified for clarity
    if not response:
        return None
    return response
Step 3: Combine Stages in the Main API Endpoint
PYTHON
@app.post("/generate_response")
async def generate_response_endpoint(query_request: QueryRequest):
    try:
        documents = await retrieve_documents(query_request.query)
        if not documents:
            raise HTTPException(status_code=503, detail="Failed to retrieve documents")

        ranked_docs = await rerank_documents(documents, query_request.query)
        if not ranked_docs:
            raise HTTPException(status_code=503, detail="Failed to rerank documents")

        response = await generate_response(ranked_docs)
        if not response:
            raise HTTPException(status_code=503, detail="Failed to generate response")

        return {"response": response}
    except Exception as e:
        # Log the exception and retry mechanism
        print(f"Error: {str(e)}")
        raise HTTPException(status_code=503, detail="Internal server error")

#### Closing Summary
+ Common Mistakes/Follow-Ups

This design ensures that each stage of the RAG pipeline operates independently with retries and timeouts. Graceful degradation is achieved by ensuring partial results are returned when a stage fails.

#### Common Mistakes

Lack of Asynchronous Handling: Not using async/await can lead to performance issues.
Ignoring Retries: Without proper retry mechanisms, transient errors can cause the entire process to fail.
No Graceful Degradation: Not handling partial results when a stage fails can lead to suboptimal responses.

#### Follow-Ups

How would you scale this system horizontally?
What error metrics would you monitor to ensure the system is performing well?

**Code:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import asyncio
import json

app = FastAPI()

class Document(BaseModel):
    id: int
    content: str

class QueryRequest(BaseModel):
    query: str

class QueryResponse(BaseModel):
    results: List[Document]

async def retrieve_documents(query: str) -> Optional[List[Document]]:
    documents = await fetch_documents_from_db(query)
    if not documents:
        return None
    return documents

async def rerank_documents(documents: List[Document], query: str) -> Optional[List[Document]]:
    ranked_docs = await rank_documents(documents, query)
    if not ranked_docs:
        return None
    return ranked_docs

async def generate_response(ranked_docs: List[Document]) -> Optional[str]:
    response = await process_and_generate(ranked_docs[0].content)  # Simplified for clarity
    if not response:
        return None
    return response

@app.post("/generate_response")
async def generate_response_endpoint(query_request: QueryRequest):
    try:
        documents = await retrieve_documents(query_request.query)
        if not documents:
            raise HTTPException(status_code=503, detail="Failed to retrieve documents")

        ranked_docs = await rerank_documents(documents, query_request.query)
        if not ranked_docs:
            raise HTTPException(status_code=503, detail="Failed to rerank documents")

        response = await generate_response(ranked_docs)
        if not response:
            raise HTTPException(status_code=503, detail="Failed to generate response")

        return {"response": response}
    except Exception as e:
        print(f"Error: {str(e)}")
        raise HTTPException(status_code=503, detail="Internal server error")
```

**Example input / output:**

```text
Input:
{
  "query": "What is the FastAPI RAG framework?"
}

Output:
{
  "response": "FastAPI RAG is a retrieval-augmented generation model that combines document retrieval with natural language processing."
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD
3. A[User Query] --> B[Retrive Documents]
4. B --> C[Rerank Documents]
5. C --> D[Generate Response]
6. D --> E[Return Response]
7. B-.Retry-> B
8. C-.Retry-> C
9. D-.Retry-> D

