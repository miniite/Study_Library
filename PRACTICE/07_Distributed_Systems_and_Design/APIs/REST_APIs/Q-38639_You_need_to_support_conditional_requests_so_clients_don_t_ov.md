---
type: "practice"
difficulty: "Deep"
---

# You need to support conditional requests so clients don't overwrite each other's edits in a collaborative document API. Walk through the If-Match, ETag, and conflict-resolution flow and what the client does on a 412.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HTTP Caching and ETags](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To support conditional requests in a collaborative document API, we need to ensure that clients can safely edit documents without overwriting each other's changes. This involves using HTTP headers like If-Match and ETag. Let’s break down the flow and how it works from first principles.

#### Triaging the Question

The question is asking us to explain the conditional request mechanism, specifically focusing on If-Match and ETag, and how a client should handle a 412 Precondition Failed response. The key concepts are understanding ETags (Entity Tags) for caching and versioning documents, using If-Match headers to ensure that only certain versions of a document are modified, and gracefully handling the failure case.

#### Assumptions
We are working with an HTTP REST API where documents can be updated by clients.
Clients will send requests to update a specific document.
The server maintains ETags for each version of the document.
Each client request includes relevant headers to ensure safe updates.

#### First Principles Solution

Document Versioning and Tagging:

The server generates an ETag when a document is created or updated. This tag uniquely identifies the current state of the document.
When a client requests to update a document, they include the If-Match header with their desired ETag value.

Client Request and Server Validation:

The client sends an HTTP PUT request to modify a specific document URL.
The server checks if the provided If-Match header matches the current ETag of the document.
If they match, the update is allowed.
If they do not match (indicating another client has modified the document), the server responds with 412 Precondition Failed.

Handling the 412 Response:

The client receives a 412 response and should gracefully handle this scenario by either retrying with a new version of the document or informing the user that their update was not successful.
Detailed Flow

Client Initiation:

HTTP
PUT /documents/123 HTTP/1.1
Host: example.com
If-Match: "etag1234"
Content-Type: application/json
Content-Length: 56

{"title": "Updated Title"}

Server Processing:

The server fetches the current ETag of document 123.
Checks if the provided If-Match header ("etag1234") matches the current ETag.
If they match, updates the document and returns a successful response (e.g., 200 OK).
If they do not match, responds with 412 Precondition Failed.
Client Response Handling
The client receives a 412 response:
HTTP
HTTP/1.1 412 Precondition Failed
Content-Type: application/json

{"error": "Document version mismatch", "currentETag": "etag5678"}
The client should then retrieve the latest document and its ETag, update their local copy, and retry the PUT request with the new If-Match header.

#### Closing Summary

By using conditional requests via If-Match and ETag, we can ensure that multiple clients editing a document do not overwrite each other's changes. This mechanism is crucial in collaborative environments where documents are frequently edited by different users. Handling 412 Precondition Failed responses allows the client to reconcile with the latest server state, ensuring data integrity.

#### Common Mistakes
and Follow-Ups
Mistake 1: Clients not handling 412 responses properly.

#### Follow-Up
Ensure clients have robust error handling logic to retry updates or inform users.
Mistake 2: Failing to check for ETag mismatches in all update requests.

#### Follow-Up
Implement validation logic at the server and client side to ensure If-Match checks are always performed.

**Comparison:**

| Step | Action | Condition |
|------|--------|-----------|
| 1    | Fetch document ETag | If-Match header matches current ETag |
| 2    | Update document in database | Only if ETags match |
| 3    | Return success response | If update is successful |
| 4    | Respond with error message | Otherwise, return 412 and current ETag |

**Code:**

```python
# Example Python code to handle If-Match precondition checks

def update_document(doc_id, new_data, current_etag):
    try:
        # Fetch document from database with ETag
        document = get_document_with_etag(doc_id)

        if document.etag == current_etag:
            # Update the document if ETags match
            updated_document = update_document_in_database(doc_id, new_data)
            return {"status": "success", "document": updated_document}
        else:
            raise ValueError("ETag mismatch")
    except ValueError as e:
        return {"error": str(e)}
```

**Example input / output:**

```text
# Initial request from client
PUT /documents/123 HTTP/1.1
Host: example.com
If-Match: "etag1234"
Content-Type: application/json
Content-Length: 56

{"title": "Updated Title"}

# Server response with ETag mismatch
HTTP/1.1 412 Precondition Failed
Content-Type: application/json

{"error": "Document version mismatch", "currentETag": "etag5678"}
```

**Data:**

```json
{
  "document": {
    "id": "123",
    "title": "Original Title"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client sends PUT request
4. If-Match header matchescurrent ETag?
5. Server updates document
6. Return 200 OK response toclient
7. Server responds with 412Precondition Failed

