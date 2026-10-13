---
type: "practice"
difficulty: "Deep"
---

# Your RAG system writes chunk embeddings and the source document store in two separate writes, and a crash between them leaves an embedding pointing at a deleted document. Explain this dual-write hazard and how the transactional-outbox pattern would fix the drift.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dead Letter Queues and Outbox](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Dead_Letter_Queues_and_Outbox.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Read-As-You-Go (RAG) system, chunks of documents are broken down into embeddings which are then stored in a document store and an embedding store. If these two writes happen as separate operations, a crash between them can result in inconsistencies—specifically, an embedding pointing to a deleted or outdated source document. The transactional-outbox pattern addresses this issue by ensuring that both writes are treated atomically.

To better understand the problem, let's break it down step-by-step:

Problem Statement: During the processing of documents, each chunk is embedded and stored in two different systems: an embedding store (e.g., vector database) and a source document store (e.g., file system or distributed storage). If a crash occurs between these two writes, one may succeed while the other fails, leading to an inconsistency.

#### Assumptions
We assume that both the embedding write and the document write are idempotent operations but not atomic. A crash can cause one operation to fail without the other.

To address this hazard, we can use the transactional-outbox pattern as follows:

Step-by-Step Solution Using Transactional-Outbox Pattern
Capture Outbox: Upon processing a document chunk, instead of directly writing both the embedding and the source document, first write the necessary data to an "outbox" or temporary storage.
Transaction Commit: Once the outbox has been populated with all required writes (embedding and source document), commit these changes atomically.
Error Handling: If any part of this process fails, roll back all changes in the outbox.

#### Example
Implementation

Let's consider a simplified example where we have an embedding store EmbeddingStore and a document store DocumentStore.

PYTHON
class TransactionalOutbox:
    def __init__(self):
        self.embeddings = []
        self.documents = []

    def add_embedding(self, chunk_id, embedding):
        self.embeddings.append((chunk_id, embedding))

    def add_document(self, chunk_id, path_to_document):
        self.documents.append((chunk_id, path_to_document))

    def commit(self):
        for chunk_id, embedding in self.embeddings:
            EmbeddingStore.store_embedding(chunk_id, embedding)

        for chunk_id, path_to_document in self.documents:
            DocumentStore.store_document(chunk_id, path_to_document)

# Usage
outbox = TransactionalOutbox()

chunk_id = "12345"
embedding = [0.1, 0.2, 0.3]  # Example embedding

outbox.add_embedding(chunk_id, embedding)
outbox.add_document(chunk_id, "/path/to/document.txt")

try:
    outbox.commit()
except Exception as e:
    print(f"Error committing transaction: {e}")
    # Rollback logic here
Advantages and Trade-offs
Atomicity: Ensures that both the embedding and document writes are treated atomically, preventing inconsistencies.
Simplicity: Simplifies error handling by ensuring all operations are managed in a single commit.
Scalability: The pattern can be extended to handle more complex transactions as needed.

#### Common Mistakes
Failing to capture the state of both writes before committing.
Not implementing rollback logic for errors, leading to partial failures.
Overlooking idempotency issues if operations are not properly designed.

