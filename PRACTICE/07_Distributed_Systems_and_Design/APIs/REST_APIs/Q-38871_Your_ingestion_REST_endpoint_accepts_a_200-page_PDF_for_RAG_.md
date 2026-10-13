---
type: "practice"
difficulty: "Deep"
---

# Your ingestion REST endpoint accepts a 200-page PDF for RAG. Chunking and embedding take minutes and the client can't wait. Design the async ingestion contract including progress reporting and how the client knows which chunks failed.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partial Failure in Fan-out APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Partial_Failure_in_Fan-out_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an asynchronous ingestion contract for a REST API that accepts a 200-page PDF, we need to ensure efficient chunking, embedding, error handling, and progress reporting. The client should receive real-time updates on which chunks have been processed successfully or failed, and the overall status of the request.

Firstly, let's clarify what the question is asking: We need to design an API endpoint that processes a large PDF file asynchronously while providing detailed feedback to the client. This includes:

Splitting the PDF into manageable chunks.
Processing each chunk through embeddings and other necessary steps.
Reporting progress and any errors encountered during processing.

#### Assumptions

The client will make a POST request with a PDF file.
We need to handle large files efficiently, likely using streaming techniques.
Errors in embedding or processing should be tracked and reported back to the client.
The system must ensure idempotency for retries.

Step-by-step reasoning from first principles:

API Design:

Create an endpoint /api/v1/ingest/pdf that accepts a POST request with the PDF file.
Use HTTP status codes to indicate success or failure.

Request Schema:

JSON
{
  "file": "<base64_encoded_pdf>"
}

Response Schema for Initial Request:

JSON
{
  "request_id": "unique-id-123",
  "status": "accepted",
  "total_chunks": 200,
  "estimated_time": "5 minutes"
}

Progress Reporting:

The client should poll an endpoint /api/v1/ingest/pdf/{request_id} to check the status.
Each chunk processing can be tracked by sending a POST request with chunk_id and status.

Error Handling:

If there is an error during embedding or processing, report it along with the relevant chunk ID.

Success Response (once all chunks are processed):

JSON
{
  "request_id": "unique-id-123",
  "total_chunks": 200,
  "successful_chunks": 195,
  "failed_chunks": [
    {"chunk_id": 5, "error_message": "Embedding failed due to data corruption"}
  ]
}

#### Closing Summary
The asynchronous ingestion contract should handle large PDF files by dividing them into smaller chunks and processing each chunk independently. The client will receive detailed progress updates and be informed of any failures. This design ensures efficient use of resources and timely feedback.

#### Common Mistakes/Follow-Ups

Not handling errors gracefully.
Overcomplicating the request-response cycles without necessary validation.
Failing to provide sufficient context for failed chunks.

**Comparison:**

```
Request ID	Status	Total Chunks	Estimated Time
unique-id-123	accepted	200	5 minutes
```

**Code:**

```python
from flask import Flask, request, jsonify
import uuid

app = Flask(__name__)

class IngestionManager:
    def __init__(self):
        self.requests = {}

    def handle_request(self, file_data):
        request_id = str(uuid.uuid4())
        total_chunks = len(file_data) // 1024  # Simplified chunking
        self.requests[request_id] = {
            "status": "accepted",
            "total_chunks": total_chunks,
            "estimated_time": "5 minutes"
        }
        return jsonify({"request_id": request_id, "total_chunks": total_chunks, "estimated_time": "5 minutes"})

    def handle_progress(self, request_id, chunk_id, status):
        if request_id in self.requests:
            self.requests[request_id]["chunks"][chunk_id] = status
            # Report back to client or log the progress

ingestion_manager = IngestionManager()

@app.route('/api/v1/ingest/pdf', methods=['POST'])
def ingest_pdf():
    file_data = base64.b64decode(request.form.get('file'))
    response = ingestion_manager.handle_request(file_data)
    return response
```

**Example input / output:**

```

POST /api/v1/ingest/pdf

JSON
Copy
{
  "file": "<base64_encoded_pdf>"
}

Response:

JSON
Copy
{
  "request_id": "unique-id-123",
  "status": "accepted",
  "total_chunks": 200,
  "estimated_time": "5 minutes"
}

Polling Progress: POST /api/v1/ingest/pdf/{request_id}

JSON
Copy
{"chunk_id": 5, "status": "processing"}

Response:

JSON
Copy
{
  "request_id": "unique-id-123",
  "status": "processing",
  "total_chunks": 200,
  "chunks": {
    "5": {"status": "failed", "error_message": "Embedding failed due to data corruption"}
  }
}
```

**Data:**

```json
{
  "request_id": "unique-id-123",
  "status": "accepted",
  "total_chunks": 200,
  "estimated_time": "5 minutes"
}
```

**Flow:**

1. Client sends PDF
2. Ingestion Manager receivesrequest
3. Generate Request ID
4. Return acceptance with totalchunks and estimated time
5. Client polls progress endpoint
6. Poll Ingestion Manager forchunk status updates or errors

