---
type: "practice"
difficulty: "Deep"
---

# How would you handle a FastAPI batch scoring endpoint that must process a 2GB uploaded file of records without loading it all into memory and while streaming partial progress back?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ File Uploads and Inference](<../../../../TOOLS/Python/FastAPI/File_Uploads_and_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle a FastAPI batch scoring endpoint that processes a 2GB uploaded file without loading it all into memory, we need to leverage streaming techniques both on the client side (upload) and the server side (processing). This ensures efficient resource utilization and allows for real-time feedback. The core idea is to break down the file into manageable chunks and process them one by one.

#### Triaging the Question

The question is primarily about implementing an endpoint in FastAPI that can handle large files efficiently, without causing memory issues. Specifically, it asks us to ensure that the file is processed in a streaming manner, allowing for partial progress updates back to the client. The interviewer is probing our understanding of:

Streaming techniques.
File handling and chunking.
Asynchronous processing.
Performance optimization.

#### First-Principles Solution

Firstly, we need to define an endpoint that accepts a file upload in chunks. FastAPI can be configured to handle multipart requests, allowing for large files to be uploaded piece by piece. On the server side, each chunk will be processed individually and partial results will be returned to the client. This approach ensures that memory usage remains low.

Step-by-Step Solution

Define the Endpoint: We define a FastAPI endpoint that can accept multipart form data containing file uploads.

Chunking the File: The file is uploaded in chunks, and each chunk is processed immediately upon receipt.

Processing Chunks Asynchronously: Each chunk is passed to an asynchronous function for processing, ensuring non-blocking execution.

Feedback Mechanism: Partial progress or results are sent back to the client after processing each chunk.

#### Implementation
Details

Here's a simplified implementation of such an endpoint using FastAPI:

PYTHON
from fastapi import APIRouter, File, UploadFile, Form
from typing import List
import asyncio

router = APIRouter()

async def process_chunk(chunk: bytes):
    # Simulate some processing (e.g., scoring)
    await asyncio.sleep(0.1)  # Simulated delay for processing
    return f"Processed chunk of {len(chunk)} bytes"

@router.post("/batch-score")
async def batch_score(file: UploadFile = File(...), data: List[bytes] = Form([])):
    if file.content_type != "application/octet-stream":
        return {"error": "Invalid file type"}

    # Stream the file in chunks
    async for chunk in file.stream(chunk_size=1024 * 1024):  # 1MB chunks
        result = await process_chunk(chunk)
        data.append(result)  # Store partial results

    return {"results": data}

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that the file is processed in manageable chunks, preventing memory overflow. By processing each chunk asynchronously, we maintain responsiveness and can provide feedback to the client during the process.

#### Common Mistakes

Not using asynchronous processing, leading to potential blocking issues.
Failing to handle edge cases like invalid file types or large chunks.
Neglecting to manage partial results properly, which could lead to data inconsistencies.

For a deeper understanding, follow-ups might include discussing how to implement retries for failed chunks, optimizing chunk size based on network conditions, and integrating with a database or storage system for persistent data handling.

