---
type: "practice"
difficulty: "Intermediate"
---

# How would you attach metadata like a source URL to each vector so your FastAPI endpoint can return it with search results?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of attaching metadata such as a source URL to vectors returned by a FastAPI endpoint, we need to ensure that our system correctly includes this information in the API response. This involves modifying how we store and retrieve vector data, ensuring our database schema supports it, and updating our FastAPI code to properly format the responses.

First, let’s break down the problem:

We need to store vectors along with their metadata.
When queried via an endpoint, these vectors should come back with the associated metadata included in the response.

#### Triaging the Question

The core of this question is about integrating additional data (metadata) into our vector search results. The interviewer might be probing your understanding of:

FastAPI basics and its architecture.
How to extend standard data models to include extra fields.
Proper handling of database queries in a web API context.

#### Assumptions
We are using a vector store (like Redis or an embedding service) for storing vectors.
Our backend is built with FastAPI, and we have a way to query this storage.
The source URL metadata is stored somewhere accessible by our application, e.g., in a PostgreSQL database.

#### Solution
from First Principles

Database Schema: We need to extend our existing vector data schema to include the metadata fields (e.g., source_url).

Data Storage: Ensure our vector store can handle additional metadata. For example, we might store vectors and their metadata in separate tables or documents.

API Endpoints: Modify our FastAPI endpoints to fetch both the vectors and their associated metadata when performing searches.

Step-by-Step Solution
1. Database Schema

Let’s assume we are using a PostgreSQL database for storing vector data along with metadata:

SQL
CREATE TABLE search_vectors (
    id SERIAL PRIMARY KEY,
    vector_embedding BYTEA NOT NULL,
    metadata JSONB NOT NULL
);

In this schema, vector_embedding holds the vector data, and metadata is a JSONB field where we can store any additional information like source URLs.

2. Data Storage in FastAPI

Suppose our vector search function returns a list of vectors:

PYTHON
from fastapi import FastAPI, HTTPException
import psycopg2
from pydantic import BaseModel

app = FastAPI()

class VectorMetadata(BaseModel):
    id: int
    source_url: str

def get_vectors_with_metadata(query: str) -> List[VectorMetadata]:
    # Dummy implementation for illustration
    result = [
        {"id": 1, "source_url": "https://example.com/source1"},
        {"id": 2, "source_url": "https://example.com/source2"}
    ]
    return result

@app.get("/search", response_model=List[VectorMetadata])
async def search_vectors(query: str):
    try:
        vectors = get_vectors_with_metadata(query)
        # Additional logic to fetch vector embeddings
        return vectors
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

In this example, get_vectors_with_metadata is a placeholder function that would ideally query the database to retrieve both vectors and their metadata.

3. Integrating with Vector Store

If we are using an external vector store like Pinecone or Weaviate, we would need to extend our queries to fetch the metadata alongside the vectors:

PYTHON
import pinecone

# Initialize Pinecone client
pinecone.init(api_key="YOUR_API_KEY", environment="us-west1-gcp")
index = pinecone.Index("your-index-name")

def get_vectors_with_metadata(query: str) -> List[VectorMetadata]:
    # Query vector store to get vectors
    results = index.query(queries=[query], top_k=5)

    # Fetch metadata from database for the retrieved IDs
    conn = psycopg2.connect(
        host="your-host",
        port="5432",
        user="your-user",
        password="your-password",
        dbname="your-dbname"
    )
    with conn.cursor() as cur:
        cur.execute("SELECT id, source_url FROM search_vectors WHERE id IN %s", (tuple([r.metadata['id'] for r in results]),))
        metadata = [VectorMetadata(**row) for row in cur.fetchall()]

    return metadata

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that the vector search results include relevant metadata. The key points are:

Extending the database schema to support additional fields.
Modifying API queries to handle both vectors and metadata.
Carefully handling data retrieval from multiple sources (e.g., external vector store and local database).

#### Common mistakes
could be forgetting to update the schema, not properly handling exceptions in data retrieval, or failing to format the JSON response correctly.

#### Follow-ups
might include:

How you would optimize this for very large datasets.
Ensuring data consistency between different data stores.
Security concerns around exposing metadata sources.

**Code:**

```python
from fastapi import FastAPI, HTTPException
import psycopg2
from pydantic import BaseModel

app = FastAPI()

class VectorMetadata(BaseModel):
    id: int
    source_url: str

def get_vectors_with_metadata(query: str) -> List[VectorMetadata]:
    result = [
        {"id": 1, "source_url": "https://example.com/source1"},
        {"id": 2, "source_url": "https://example.com/source2"}
    ]
    return result

@app.get("/search", response_model=List[VectorMetadata])
async def search_vectors(query: str):
    try:
        vectors = get_vectors_with_metadata(query)
        # Additional logic to fetch vector embeddings
        return vectors
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

**Example input / output:**

```

Request to /search?q=some-query:

Input: {"query": "some-query"}
Output:
JSON
Copy
[
    {"id": 1, "source_url": "https://example.com/source1"},
    {"id": 2, "source_url": "https://example.com/source2"}
]
```

**Data:**

```sql
CREATE TABLE search_vectors (
    id SERIAL PRIMARY KEY,
    vector_embedding BYTEA NOT NULL,
    metadata JSONB NOT NULL
);
```

