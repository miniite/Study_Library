---
type: "practice"
difficulty: "Intermediate"
---

# Why is document ingestion for RAG usually handled as a separate REST endpoint from the actual question-answering endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Partial Failure in Fan-out APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Partial_Failure_in_Fan-out_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of Retrieval-Augmented Generation (RAG) systems, document ingestion and question-answering are two distinct processes that serve different purposes and have different performance requirements. Typically, document ingestion is handled as a separate REST endpoint to ensure scalability, efficiency, and maintainability of the system.

Firstly, the primary purpose of a document ingestion endpoint is to receive and preprocess large amounts of unstructured text data (like PDFs, articles, or other documents). This process involves tasks such as extracting content, normalizing formats, tokenizing, and indexing. These operations are computationally intensive and can take significant time to complete, often involving external services like NLP tools for text processing.

On the other hand, a question-answering (QA) endpoint is designed to respond quickly to user queries by leveraging preprocessed documents stored in an index. The goal here is to provide fast, accurate answers using the already processed content. Keeping these two processes separate allows for efficient management of resources and ensures that users do not experience delays due to time-consuming document preparation tasks.

Key aspects to consider when designing this separation include:

Scalability: Document ingestion can be resource-intensive and may require specialized hardware or cloud services, whereas QA can handle multiple concurrent requests with minimal overhead.
Error Handling: Ingestion failures (e.g., due to malformed documents) should not affect the availability of the QA service. Separate endpoints allow for robust error handling without impacting user experience.
Performance Optimization: By isolating these functions, developers can optimize each process independently for maximum efficiency.

Let's break down the triage and reasoning from first principles:

Understanding the Core Problem:

The question is asking why document ingestion is usually handled separately in RAG systems.
The core of the issue revolves around operational efficiency, resource management, and user experience.

#### Assumptions

We are assuming a typical RAG system architecture where both ingestion and QA processes need to be efficient.
There may be different levels of service requirements for each process (e.g., real-time vs batch processing).

Step-by-Step Solution:

Document Ingestion Endpoint:
Receive the document data via a REST POST request.
Preprocess and extract text content using NLP tools.
Normalize and tokenize the text for indexing.
Index the processed documents in a database or search engine.
Question-Answering Endpoint:
Receive user queries via a REST GET request.
Use pre-indexed documents to generate relevant answers quickly.

By separating these processes, we ensure that ingestion can be performed offline or as a batch process without affecting the real-time QA service. This separation also allows for better resource allocation and fault isolation.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
The key takeaway is that document ingestion and question-answering are fundamentally different processes, each requiring distinct design considerations.
A common mistake might be conflating these two processes, leading to suboptimal performance or scalability issues.

#### Follow-ups
could explore specific implementations of such endpoints in real-world systems.

**Comparison:**

```
Step	Document Ingestion	Question-Answering
Time Complexity	High	Low
Resource Intensive	Yes	No
```

**Code:**

```python
# Example code for document ingestion endpoint

@app.route('/ingest', methods=['POST'])
def ingest_document():
    file = request.files['document']
    if not file:
        return jsonify({"error": "No file provided"}), 400

    content = process_file(file)
    index_document(content)

    return jsonify({"message": "Document ingested successfully"}), 201
```

**Example input / output:**

```text
Input:
- POST /ingest
- Request body: {"document": {"filename": "example.pdf", "content_type": "application/pdf"}}

Output:
- Response status code: 201 (Created)
- Response body: {"message": "Document ingested successfully"}
```

**Data:**

```json
{
  "document": {
    "filename": "example.pdf",
    "content_type": "application/pdf"
  }
}
```

**Flow:**

1. Yes
2. Yes
3. Ingestion Endpoint
4. File Received?
5. Preprocess Document
6. Index Document
7. Return Success Response
8. QA Endpoint
9. Query Received?
10. Retrieve Relevant Documents
11. Generate Answer
12. Return Answer

