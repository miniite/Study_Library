---
type: "practice"
difficulty: "Intermediate"
---

# In a RAG system, what REST calls happen between the user asking a question and the LLM producing an answer? Walk through the sequence.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Partial Failure in Fan-out APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Partial_Failure_in_Fan-out_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the sequence of REST API calls in a Retrieval-Augmented Generation (RAG) system from when a user asks a question to when the language model (LLM) produces an answer, we need to break down the process step by step. This involves multiple REST calls that handle retrieval of relevant documents, preprocessing these documents, feeding them into the LLM for generation, and finally returning the generated response back to the user.

#### Triage
What the Question is Asking

The question probes understanding of how a RAG system works at an API level. It requires explaining the flow from the initial request made by the user to the LLM generating a response. The key points we need to cover are:

Retrieval of relevant documents.
Preprocessing these documents for input into the LLM.
Feeding preprocessed data into the LLM for generation.
Returning the generated answer back to the user.

#### Assumptions
We assume there's a REST API endpoint available for interacting with both the retrieval and generation processes.
The system has an index of documents that can be searched based on the user’s query.
Preprocessing involves cleaning, tokenizing, or embedding the text before sending it to the LLM.

#### First-Principles Solution
User Request:
User sends a GET request to /api/search with their question as a parameter.
Document Retrieval:
The system makes a GET call to an internal document retrieval API at /api/document/retrieve, passing the user’s query as a parameter.
Preprocessing Documents:
Once relevant documents are retrieved, they might go through preprocessing steps like cleaning and tokenizing. This can be done internally or by another service.
Feeding Preprocessed Data to LLM:
The preprocessed text is then sent via POST request to /api/llm/generate, where the LLM processes it to generate a response.
Generating Response:
The LLM generates an answer based on the preprocessed input and returns this response.
Returning Response to User:
Finally, the generated response is sent back to the user via another GET or POST request to /api/response.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The core of the process involves making REST calls at each step to handle document retrieval, preprocessing, and finally generating the response. A common mistake is not properly handling errors in each step (e.g., no documents found for a query) or missing necessary preprocessing steps before sending data to the LLM.

For a more robust solution:

Implement error handling and validation.
Use caching mechanisms to improve performance during document retrieval.
Optimize preprocessing pipelines to ensure efficiency.

#### Example
Input-Output
PLAINTEXT
User requests: GET /api/search?query=AI engineering challenges

API response (Document Retrieval):
GET /api/document/retrieve?query=AI engineering challenges
Response:
[{"id": 1, "title": "Engineering Challenges in AI", "text": "This article discusses the..."}, {"id": 2, "title": "..."}]

Preprocessing Documents:
Clean and tokenize text for LLM input.

Generate Response:
POST /api/llm/generate?documents=[cleaned_documents]
Response:
{"answer": "AI engineering challenges include data quality issues, model interpretability,..."}

Return to User:
GET /api/response
Response:
{"answer": "AI engineering challenges include data quality issues, model interpretability,..."}

**Comparison:**

```
Step	REST Call	Parameters	Response
1	GET	/search	Query
2	GET	/retrieve	Documents
3	N/A	Preprocessing	Cleaned Docs
4	POST	/generate	Answer
5	GET/POST	/response	Response
```

**Code:**

```python
# Example Python Code for Document Retrieval and Preprocessing

import requests

def retrieve_documents(query):
    response = requests.get(f"http://document-service/api/document/retrieve?query={query}")
    documents = response.json()
    return [doc['text'] for doc in documents]

def preprocess_documents(documents):
    # Simple preprocessing example
    cleaned_documents = [doc.lower() for doc in documents]
    return cleaned_documents

documents = retrieve_documents("AI engineering challenges")
preprocessed_docs = preprocess_documents(documents)
```

**Example input / output:**

```text
User request: GET /api/search?query=AI engineering challenges

Document Retrieval API response:
GET /api/document/retrieve?query=AI engineering challenges
Response:
[{"id": 1, "title": "Engineering Challenges in AI", "text": "This article discusses the..."}, {"id": 2, "title": "..."}]

Preprocessing Documents:
Clean and tokenize text for LLM input.

Generate Response:
POST /api/llm/generate?documents=[cleaned_documents]
Response:
{"answer": "AI engineering challenges include data quality issues, model interpretability,..."}

Return to User:
GET /api/response
Response:
{"answer": "AI engineering challenges include data quality issues, model interpretability,..."}
```

**Data:**

```json
[
  {"id": 1, "title": "Engineering Challenges in AI", "text": "This article discusses the various..."},
  {"id": 2, "title": "..."}
]
```

**Flow:**

1. Yes
2. Yes
3. No
4. User submits query
5. Is document retrievalsuccessful?
6. Retrieve relevant documents
7. Preprocess documents
8. Pass processed data to LLM forgeneration
9. LLM generates response
10. Does the user request aresponse?
11. Return generated response touser
12. Error handling and logging

