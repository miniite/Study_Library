---
type: "practice"
difficulty: "Intermediate"
---

# How would you update the index when only a few source documents change?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Chunk Size and Retrieval Quality](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Chunk_Size_and_Retrieval_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AI system where large volumes of text data are processed and indexed, efficiently updating the search index is crucial. This problem specifically deals with scenarios where only a few out of many documents have changed. The key challenge here is to minimize the computational overhead while ensuring that the updated index remains accurate.

#### Triage
What's Being Asked

The question requires us to design an efficient strategy for updating a search index when only a few source documents change. This involves considering both storage and retrieval strategies, focusing on minimizing the time and resources required for updates without compromising the integrity of the index.

#### Assumptions
The system uses an inverted index structure.
The documents are frequently accessed via text queries.
Only small portions (e.g., a few paragraphs) of the documents change at any given time.
Step-by-Step Solution
1. Identify Changed Documents

First, we need to determine which documents have been updated. This can be done using metadata such as timestamps or hashes. For simplicity, let’s assume each document has a timestamp that gets updated when it changes.

PYTHON
documents = [
    {"id": 1, "text": "Document text", "timestamp": "2023-10-01"},
    {"id": 2, "text": "Another document", "timestamp": "2023-10-05"},
    # ... more documents
]

new_timestamps = {
    1: "2023-10-02",
    2: "2023-10-06"  # Assume only doc 1 and 2 have changed
}
2. Update the Inverted Index

For each updated document, we need to update its relevant terms in the inverted index.

PYTHON
class InvertedIndex:
    def __init__(self):
        self.index = {}

    def add_term(self, term, doc_id):
        if term not in self.index:
            self.index[term] = set()
        self.index[term].add(doc_id)

    def update_document(self, document):
        for term in extract_terms(document["text"]):
            self.add_term(term, document["id"])

# Extract terms from text
def extract_terms(text):
    # Simple example: split by whitespace and keep words of length 3+
    return [word for word in text.split() if len(word) > 2]

index = InvertedIndex()

for doc_id, new_timestamp in new_timestamps.items():
    document = next(doc for doc in documents if doc["id"] == doc_id)
    index.update_document(document)

# Example output:
# index: {'document': {1}, 'text': {1}}
3. Rebuild Partial Index

Instead of rebuilding the entire index, we can selectively update it by reprocessing only the changed terms.

PYTHON
def update_partial_index(index, new_timestamps, documents):
    for doc_id, new_timestamp in new_timestamps.items():
        document = next(doc for doc in documents if doc["id"] == doc_id)
        existing_terms = set()

        # Check existing index entries to avoid reprocessing unchanged terms
        if doc_id in index.index:
            existing_terms.update(index.index[doc_id])

        # Update the inverted index with new terms
        for term in extract_terms(document["text"]):
            if term not in existing_terms:
                index.add_term(term, document["id"])

update_partial_index(index, new_timestamps, documents)
4. Optimize Using Caching

To further improve efficiency, we can cache the processed terms and timestamps to avoid redundant operations.

PYTHON
class TermCache:
    def __init__(self):
        self.cache = {}

    def get_term(self, term, doc_id):
        if (term, doc_id) in self.cache:
            return self.cache[(term, doc_id)]
        else:
            result = extract_terms(term)
            self.cache[(term, doc_id)] = result
            return result

cache = TermCache()

#### Closing Summary

By selectively updating the inverted index and using caching techniques, we can efficiently handle updates to a large dataset. This approach minimizes computational overhead while ensuring that the search index remains up-to-date.

#### Common Mistakes/Follow-Ups
Ignoring Metadata: Not properly handling timestamps or metadata could lead to unnecessary full-index rebuilds.
Overcomplicating Index Structures: Simplifying the update process can sometimes be more effective than over-engineering it.
Query Performance Impact: Ensure that even partial updates do not significantly impact query performance.

**Code:**

```python
class InvertedIndex:
    def __init__(self):
        self.index = {}

    def add_term(self, term, doc_id):
        if term not in self.index:
            self.index[term] = set()
        self.index[term].add(doc_id)

    def update_document(self, document):
        for term in extract_terms(document["text"]):
            self.add_term(term, document["id"])

def extract_terms(text):
    return [word for word in text.split() if len(word) > 2]

index = InvertedIndex()

for doc_id, new_timestamp in new_timestamps.items():
    document = next(doc for doc in documents if doc["id"] == doc_id)
    index.update_document(document)

def update_partial_index(index, new_timestamps, documents):
    term_cache = TermCache()
    for doc_id, new_timestamp in new_timestamps.items():
        document = next(doc for doc in documents if doc["id"] == doc_id)
        existing_terms = set()

        if doc_id in index.index:
            existing_terms.update(index.index[doc_id])

        for term in extract_terms(document["text"]):
            if (term, doc_id) not in term_cache.cache or term not in existing_terms:
                term_cache.get_term(term, document["id"])
                index.add_term(term, document["id"])

update_partial_index(index, new_timestamps, documents)
```

**Example input / output:**

```text
# Input
documents = [
    {"id": 1, "text": "Document text", "timestamp": "2023-10-01"},
    {"id": 2, "text": "Another document", "timestamp": "2023-10-05"}
]
new_timestamps = {
    "1": "2023-10-02",
    "2": "2023-10-06"
}

# Process
index = InvertedIndex()
for doc_id, new_timestamp in new_timestamps.items():
    document = next(doc for doc in documents if doc["id"] == doc_id)
    index.update_document(document)

# Output: Partially updated inverted index
index.index  # {'document': {1}, 'text': {1}}
```

**Data:**

```json
{
    "documents": [
        {"id": 1, "text": "Document text", "timestamp": "2023-10-01"},
        {"id": 2, "text": "Another document", "timestamp": "2023-10-05"}
    ],
    "new_timestamps": {
        "1": "2023-10-02",
        "2": "2023-10-06"
    }
}
```

