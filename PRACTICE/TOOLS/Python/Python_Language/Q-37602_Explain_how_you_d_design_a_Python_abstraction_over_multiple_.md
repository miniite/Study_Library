---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a Python abstraction over multiple vector stores (Pinecone, pgvector, Milvus) so teams can switch backends without rewriting retrieval code, and what leaks through the abstraction that you must expose deliberately.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Functions Modules and Reuse](<../../../../TOOLS/Python/Python_Language/Functions_Modules_and_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a Python abstraction over multiple vector stores like Pinecone, pgvector, and Milvus, we need to create a unified interface that abstracts away the underlying storage details while ensuring that teams can seamlessly switch between these backends without rewriting retrieval code. The key idea is to define a common set of operations and behaviors that all backend implementations must support, thereby decoupling application logic from specific vector store implementations.

#### Triaging the Question

The question is essentially asking for an approach to build a generic vector storage abstraction in Python. We need to ensure this abstraction is flexible enough to accommodate different vector stores while providing the necessary APIs for common operations (e.g., adding vectors, querying nearest neighbors). The interviewer might be probing your understanding of design patterns, modularity, and how to manage complexity in software engineering.

#### First-Principles Solution
Define a Common Interface: Create an abstract base class or interface that defines the required methods for vector storage operations.
Implement Backend Specifics: Write concrete classes for each backend (Pinecone, pgvector, Milvus) to implement these methods.
Client Code Independence: Ensure client code interacts only with the abstract interface, making it agnostic to specific backends.
Configuration Management: Provide a mechanism to configure which backend is used at runtime.
Detailed Solution
1. Define the Common Interface
PYTHON
from abc import ABC, abstractmethod
import numpy as np

class VectorStore(ABC):
    @abstractmethod
    def add_vectors(self, vectors: list[np.ndarray], ids: list[str]) -> None:
        pass

    @abstractmethod
    def query_nearest_neighbors(self, vector: np.ndarray, k: int) -> list[tuple[str, float]]:
        pass

    @abstractmethod
    def close(self) -> None:
        pass
2. Implement Backend Specifics

Here are sample implementations for each backend:

PYTHON
class PineconeStore(VectorStore):
    # Implementation details specific to Pinecone
    ...

class PgvectorStore(VectorStore):
    # Implementation details specific to pgvector
    ...

class MilvusStore(VectorStore):
    # Implementation details specific to Milvus
    ...
3. Client Code Independence
PYTHON
def perform_query(vector_store: VectorStore, vector: np.ndarray, k: int) -> list[tuple[str, float]]:
    return vector_store.query_nearest_neighbors(vector, k)

# Example usage:
vector_store = PineconeStore()  # Or PgvectorStore, MilvusStore
perform_query(vector_store, np.array([1.0, 2.0]), 5)
4. Configuration Management
PYTHON
def create_vector_store(config: dict) -> VectorStore:
    backend = config['backend']
    if backend == 'pinecone':
        return PineconeStore()
    elif backend == 'pgvector':
        return PgvectorStore()
    elif backend == 'milvus':
        return MilvusStore()
    else:
        raise ValueError("Unsupported vector store backend")

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that the application code remains independent of the specific vector storage implementation. The VectorStore interface provides a clear contract, and each backend must conform to it. By using dynamic configuration at runtime, teams can switch between backends without changing their application logic.

#### Common Mistakes

Forgetting to define abstract methods in the base class.
Not properly handling initialization and cleanup (e.g., closing connections).
Overcomplicating the interface with too many unnecessary operations.
Not providing adequate error handling or logging.

#### Follow-ups
might include discussing how to handle vector store-specific optimizations, performance considerations, security implications, and how to extend this abstraction for other types of storage or retrieval patterns.

**Comparison:**

```
Vector Store	Backend Configuration
Pinecone	{'backend': 'pinecone'}
Pgvector	{'backend': 'pgvector'}
Milvus	{'backend': 'milvus'}
```

**Code:**

```python
from abc import ABC, abstractmethod
import numpy as np

class VectorStore(ABC):
    @abstractmethod
    def add_vectors(self, vectors: list[np.ndarray], ids: list[str]) -> None:
        pass

    @abstractmethod
    def query_nearest_neighbors(self, vector: np.ndarray, k: int) -> list[tuple[str, float]]:
        pass

    @abstractmethod
    def close(self) -> None:
        pass

class PineconeStore(VectorStore):
    # Implementation details specific to Pinecone
    ...

class PgvectorStore(VectorStore):
    # Implementation details specific to pgvector
    ...

class MilvusStore(VectorStore):
    # Implementation details specific to Milvus
    ...

def perform_query(vector_store: VectorStore, vector: np.ndarray, k: int) -> list[tuple[str, float]]:
    return vector_store.query_nearest_neighbors(vector, k)

def create_vector_store(config: dict) -> VectorStore:
    backend = config['backend']
    if backend == 'pinecone':
        return PineconeStore()
    elif backend == 'pgvector':
        return PgvectorStore()
    elif backend == 'milvus':
        return MilvusStore()
    else:
        raise ValueError("Unsupported vector store backend")
```

**Example input / output:**

```text
# Example input and output

config = {
    "backend": "milvus"
}

vector_store = create_vector_store(config)
result = perform_query(vector_store, np.array([1.0, 2.0]), 5)

print(result)  # Output: [('id1', 0.9), ('id2', 0.8)]
```

**Data:**

```json
{
  "backend": "pinecone"
}
```

**Flow:**

1. Client Code
2. CreateVectorStore
3. PineconeStore|PgvectorStore|MilvusStore
4. PerformQuery

