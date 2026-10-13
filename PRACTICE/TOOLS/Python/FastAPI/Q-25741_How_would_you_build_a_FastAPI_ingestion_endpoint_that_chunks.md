---
type: "practice"
difficulty: "Deep"
---

# How would you build a FastAPI ingestion endpoint that chunks, embeds, and upserts a large document set as a background job while reporting progress and handling partial failures?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ RAG Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a FastAPI ingestion endpoint that processes a large document set by chunking, embedding, and upserting them in a background job with progress reporting and failure handling, we need to break down the problem into several key components. This solution will leverage FastAPI for handling the HTTP requests, Celery for managing background tasks, and a database (like PostgreSQL) to store documents and their embeddings.

The core idea is to create an efficient pipeline where documents are read in chunks, processed with embeddings, and stored asynchronously using a job system that can handle failures gracefully. This ensures that even if some parts of the process fail, others can continue without interruption.

**Code:**

```python
from fastapi import FastAPI, File, UploadFile, BackgroundTasks
from fastapi.responses import JSONResponse
import os
import uuid
import logging

app = FastAPI()
logger = logging.getLogger("uvicorn")

def chunk_file(file_path, chunk_size=1024):
    with open(file_path, 'r') as f:
        while True:
            data = f.read(chunk_size)
            if not data:
                break
            yield data

def process_chunk(chunk):
    # Example embedding logic; replace with actual embeddings
    return {"chunk_id": str(uuid.uuid4()), "embedding": [random.random() for _ in range(5)]}

@app.post("/ingest")
async def ingest_document(file: UploadFile, background_tasks: BackgroundTasks):
    temp_file_path = f"/tmp/{file.filename}"

    # Save the uploaded file temporarily
    with open(temp_file_path, "wb") as buffer:
        while chunk := await file.read(1024):  # async read chunks
            buffer.write(chunk)

    # Process and upsert in background
    background_tasks.add_task(process_document_set, temp_file_path)

async def process_document_set(file_path):
    try:
        for chunk in chunk_file(file_path):
            embedding = process_chunk(chunk)

            # Simulate storing to database (e.g., PostgreSQL) with UUID for doc ID
            store_embedding_to_db(embedding)

            logger.info(f"Processed and stored: {embedding}")

    except Exception as e:
        logger.error(f"Failed processing document set: {e}")

def store_embedding_to_db(embedding):
    # Placeholder for database interaction
    pass
```

**Data:**

```json
{
  "chunk_file": {
    "description": "Function to read a file in chunks",
    "parameters": [
      {"name": "file_path", "type": "str", "description": "Path to the file to be chunked"},
      {"name": "chunk_size", "type": "int", "default": 1024, "description": "Size of each chunk in bytes"}
    ],
    "returns": "Generator[str]"
  },
  "process_chunk": {
    "description": "Function to process a chunk (e.g., generate embeddings)",
    "parameters": [
      {"name": "chunk", "type": "str", "description": "Chunk of the file content to be processed"}
    ],
    "returns": "dict"
  },
  "process_document_set": {
    "description": "Function to process and store a set of documents asynchronously",
    "parameters": [
      {"name": "file_path", "type": "str", "description": "Path to the file containing document chunks"}
    ],
    "returns": "None"
  },
  "store_embedding_to_db": {
    "description": "Function to simulate storing an embedding in a database",
    "parameters": [
      {"name": "embedding", "type": "dict", "description": "Dictionary containing chunk ID and embedding data"}
    ],
    "returns": "None"
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Document Upload
4. Is File Larger Than Threshold?
5. Split into Chunks
6. Process Each Chunk inBackground Using Celery Task
7. Directly Process and Upsert
8. Store Embedding Result inDatabase
9. Handle Success or Failure

