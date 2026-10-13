---
type: "practice"
difficulty: "Intermediate"
---

# What does over-fetching mean when your RAG endpoint returns full document chunks the client does not need, and how would you trim the response?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Partial Failure in Fan-out APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Partial_Failure_in_Fan-out_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Over-fetching in the context of a REST API means returning more data to the client than is strictly necessary or requested. In the case of a RAG (Retrieval-Augmented Generation) endpoint that returns full document chunks, over-fetching occurs when the returned documents contain redundant information that the client does not need for its current task. To address this, we can implement a mechanism to trim the response, ensuring only relevant data is sent.

In an intermediate context, understanding and implementing such optimizations involves several steps: identifying the excess data, determining what's necessary, and then crafting a streamlined API response. This process requires careful consideration of performance, storage costs, and user experience.

**Code:**

```python
def trim_response(full_document):
    # Example function to filter out unnecessary parts of the document
    if 'header' in full_document:
        del full_document['header']
    if 'footer' in full_document:
        del full_document['footer']

    return full_document

# Sample input: a dictionary representing a full document chunk
sample_document = {
    "header": {"title": "Document Title", "date": "2023-10-10"},
    "body": "This is the main content of the document.",
    "footer": {"author": "Author Name"}
}

# Trimmed response after removing unnecessary parts
trimmed_document = trim_response(sample_document)
print(trimmed_document)
```

**Example input / output:**

```

Input:

PYTHON
Copy
sample_document = {
    "header": {"title": "Document Title", "date": "2023-10-10"},
    "body": "This is the main content of the document.",
    "footer": {"author": "Author Name"}
}

Output:

PYTHON
Copy
{
    "body": "This is the main content of the document."
}
```

**Flow:**

1. Client Request
2. API Endpoints
3. RAG Endpoint
4. Fetch Full Document Chunks
5. Identify Unnecessary Parts
6. Remove Unnecessary Parts
7. Return Trimmed Response
8. Client Receives Trimmed Data

