---
type: "practice"
difficulty: "Intermediate"
---

# Your RAG citations must map each returned chunk back to its source document and offset in O(1) after a vector search returns IDs; what structure holds the chunk-to-source mapping and how do you keep it in sync on re-ingest?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Diff and LCS Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Diff_and_LCS_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that each returned chunk from a vector search can be mapped back to its source document and offset in O(1) time, we need an efficient data structure that allows us to store and quickly retrieve this information. The primary challenge is keeping the mapping up-to-date as new documents are ingested or existing ones are updated. Here’s how you can achieve this.

#### Triaging the Question

The question is asking for a data structure to hold chunk-to-source document mappings efficiently, with the ability to update it in O(1) time during re-ingest operations. The key points we need to address are:

How to store and retrieve the mapping efficiently.
How to handle updates (re-ingest) without losing performance.

#### Assumptions
Each chunk has a unique identifier from the vector search.
Documents can be added, updated, or removed during re-ingest.
We need O(1) access time for both retrieval and update operations.

#### First Principles Solution

To achieve this, we can use a combination of hash maps and linked lists. Specifically:

Hash Map: To quickly map chunk IDs to their source document and offset information.
Linked List (or Array): To maintain the order of chunks within each document for easy update operations.
Step-by-Step Solution

Chunk ID to Document Mapping:

Use a hash map where keys are chunk IDs and values are pairs containing the document ID and the offset in that document.

Document Storage:

Maintain an array or linked list of chunks for each document, allowing us to easily insert, delete, and update chunks within a document.
Code Implementation

Let's implement this with some sample code:

PYTHON
class DocumentChunkMap:
    def __init__(self):
        self.chunk_map = {}  # {chunk_id: (doc_id, offset)}
        self.doc_chunks = {}  # {doc_id: [chunk_ids]}

    def add_chunk(self, chunk_id, doc_id, offset):
        if chunk_id in self.chunk_map:
            raise ValueError("Chunk already exists")

        self.chunk_map[chunk_id] = (doc_id, offset)
        if doc_id not in self.doc_chunks:
            self.doc_chunks[doc_id] = []
        self.doc_chunks[doc_id].append(chunk_id)

    def update_chunk(self, chunk_id, new_offset):
        # Update the offset of an existing chunk
        if chunk_id not in self.chunk_map:
            raise KeyError("Chunk does not exist")

        doc_id, _ = self.chunk_map[chunk_id]
        _, old_index = zip(*[(cid, i) for cid, (d, _) in enumerate(self.doc_chunks[doc_id]) if d == doc_id and cid == chunk_id])
        self.doc_chunks[doc_id][old_index[0]] = new_offset

    def remove_chunk(self, chunk_id):
        # Remove a chunk from the map
        if chunk_id not in self.chunk_map:
            raise KeyError("Chunk does not exist")

        doc_id, _ = self.chunk_map.pop(chunk_id)
        index_to_remove = [i for i, cid in enumerate(self.doc_chunks[doc_id]) if cid == chunk_id]
        del self.doc_chunks[doc_id][index_to_remove[0]]

#### Closing Summary

The solution uses a hash map to quickly find the document and offset of any given chunk by its ID. Simultaneously, we maintain an array or linked list for each document that keeps track of chunk IDs in order, making it easy to update chunks within a document.

#### Common Mistakes/Follow-Ups
Mistake 1: Using only a hash map without any form of secondary storage (like an array) can make updates and deletions inefficient.
Mistake 2: Not handling the case where chunk IDs might collide or change, leading to incorrect mappings.

#### Follow-up
questions could include:

How would you handle concurrent access?
What if documents have very large numbers of chunks? Would your solution still perform well?

**Code:**

```python
class DocumentChunkMap:
    def __init__(self):
        self.chunk_map = {}  # {chunk_id: (doc_id, offset)}
        self.doc_chunks = {}  # {doc_id: [chunk_ids]}

    def add_chunk(self, chunk_id, doc_id, offset):
        if chunk_id in self.chunk_map:
            raise ValueError("Chunk already exists")

        self.chunk_map[chunk_id] = (doc_id, offset)
        if doc_id not in self.doc_chunks:
            self.doc_chunks[doc_id] = []
        self.doc_chunks[doc_id].append(chunk_id)

    def update_chunk(self, chunk_id, new_offset):
        # Update the offset of an existing chunk
        if chunk_id not in self.chunk_map:
            raise KeyError("Chunk does not exist")

        doc_id, _ = self.chunk_map[chunk_id]
        _, old_index = zip(*[(cid, i) for cid, (d, _) in enumerate(self.doc_chunks[doc_id]) if d == doc_id and cid == chunk_id])
        self.doc_chunks[doc_id][old_index[0]] = new_offset

    def remove_chunk(self, chunk_id):
        # Remove a chunk from the map
        if chunk_id not in self.chunk_map:
            raise KeyError("Chunk does not exist")

        doc_id, _ = self.chunk_map.pop(chunk_id)
        index_to_remove = [i for i, cid in enumerate(self.doc_chunks[doc_id]) if cid == chunk_id]
        del self.doc_chunks[doc_id][index_to_remove[0]]
```

