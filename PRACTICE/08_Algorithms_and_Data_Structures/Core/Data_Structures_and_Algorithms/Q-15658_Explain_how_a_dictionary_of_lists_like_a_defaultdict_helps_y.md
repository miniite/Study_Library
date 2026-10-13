---
type: "practice"
difficulty: "Basic"
---

# Explain how a dictionary of lists (like a defaultdict) helps you group retrieved chunks by their source document for a citations panel.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Skip Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Skip_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working on a system that needs to manage and display citations from various documents, grouping citation chunks by their source document can be efficiently achieved using a dictionary of lists (often implemented as defaultdict in Python). This structure allows us to quickly store and access all the citation information related to each document. The key idea is that every time we retrieve a chunk of citation data, we can directly associate it with its corresponding document ID, which acts as the key.

The question asks how such a dictionary helps manage grouped citation chunks by their source documents. Let’s break down the process step-by-step:

Understanding the Problem: We need to organize citations from multiple documents in such a way that all citations for each document are easily accessible and can be displayed together.

Key Assumptions:

Each citation chunk contains a document ID and other relevant information like author, date, and content.
The goal is to efficiently group these chunks by their source documents.

#### First-Principles Solution

We initialize a dictionary where the keys are the document IDs, and the values are lists of citation chunks corresponding to that document ID.
For every new chunk retrieved from a specific document, we check if it already exists in our dictionary under that document’s ID key. If not, we add a new entry with an empty list. Then, we append the current chunk to this list.

Here is a step-by-step explanation of how this works:

Initialization: Create an empty defaultdict.
Processing Each Citation Chunk:
Retrieve or process each citation chunk.
Extract the document ID from the chunk.
Check if the document ID already exists as a key in our dictionary.
If it does, append the current chunk to the list associated with that document ID.
If it doesn't, add the document ID as a new key and initialize an empty list for its value, then append the chunk.

This approach ensures efficient grouping of citations by their source documents, making the display and management of information straightforward. It leverages Python’s defaultdict to automatically handle the creation of new lists if necessary, streamlining the process.

**Comparison:**

```
Document ID	Citations
1	[Citation from John Doe, Citation from Alice Johnson]
2	[Citation from Jane Smith]

This table illustrates how the citations are grouped by their document IDs.
```

**Code:**

```python
from collections import defaultdict

# Example citation chunk structure
class CitationChunk:
    def __init__(self, doc_id, author, date, content):
        self.doc_id = doc_id
        self.author = author
        self.date = date
        self.content = content

# Initialize the dictionary
citations_by_doc = defaultdict(list)

def process_citation_chunk(chunk: CitationChunk):
    # Add or append the chunk to the corresponding list in citations_by_doc
    if chunk.doc_id not in citations_by_doc:
        citations_by_doc[chunk.doc_id] = []
    citations_by_doc[chunk.doc_id].append(chunk)
```

**Example input / output:**

```

Given the following input data:

JSON
Copy
[
  {"doc_id": 1, "author": "John Doe", "date": "2023-01-01"},
  {"doc_id": 2, "author": "Jane Smith", "date": "2023-02-01"},
  {"doc_id": 1, "author": "Alice Johnson", "date": "2023-03-01"}
]

Processing each chunk and storing them in the citations_by_doc dictionary:

PYTHON
Copy
from collections import defaultdict

# Initialize the dictionary
citations_by_doc = defaultdict(list)

# Process the chunks
for citation in citations:
    chunk = CitationChunk(**citation)
    process_citation_chunk(chunk)

print(citations_by_doc)

Output:

PYTHON
Copy
{
  1: [
    {"doc_id": 1, "author": "John Doe", "date": "2023-01-01"},
    {"doc_id": 1, "author": "Alice Johnson", "date": "2023-03-01"}
  ],
  2: [
    {"doc_id": 2, "author": "Jane Smith", "date": "2023-02-01"}
  ]
}
```

**Data:**

```json
{
  "citations": [
    {
      "doc_id": 1,
      "author": "John Doe",
      "date": "2023-01-01",
      "content": "Citation content for document 1"
    },
    {
      "doc_id": 2,
      "author": "Jane Smith",
      "date": "2023-02-01",
      "content": "Citation content for document 2"
    },
    {
      "doc_id": 1,
      "author": "Alice Johnson",
      "date": "2023-03-01",
      "content": "More citation content for document 1"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start Processing Citations
4. Is doc_id in dict?
5. Append chunk to list
6. Initialize new empty list fordoc_id
7. Append chunk
8. Citations processed
9. End

