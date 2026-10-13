---
type: "practice"
difficulty: "Basic"
---

# How would you store a corpus of document embeddings in memory so you can retrieve the i-th document's vector in O(1)?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Arrays and Embedding Lookup](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays_and_Embedding_Lookup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store a corpus of document embeddings in memory and retrieve the (i)-th document’s vector in constant time ((O(1))), we need to use an appropriate data structure that allows direct access to any index without having to traverse through all previous elements. The most straightforward solution for this is to utilize an array.

#### Triage

The question asks about storing a corpus of document embeddings and retrieving the (i)-th document’s vector in constant time. This means we need a method where each query can be answered with minimal computation, ideally just by looking up an index in an array.

#### First-Principles Solution

Understanding the Problem:

We have a corpus of documents, and for each document, there is a corresponding vector embedding.
The goal is to store these vectors such that accessing any particular (i)-th document’s vector is efficient (in (O(1)) time).

Choosing the Data Structure:

An array is ideal because it allows direct access via indexing.
Each element in the array will represent a document's embedding.

#### Implementation
Steps:

Create an array where each index corresponds to a unique identifier of a document (e.g., its ID).
Store the vector embeddings at these indices.

#### Example

Assume we have 5 documents with their respective vector embeddings as follows:

Document 1: [0.2, 0.3]
Document 2: [-0.4, 0.6]
Document 3: [0.8, -0.9]
Document 4: [0.5, 0.7]
Document 5: [-0.1, 0.2]

We can store these in an array where each element is the vector embedding:

PYTHON
embeddings = [
    [0.2, 0.3],
    [-0.4, 0.6],
    [0.8, -0.9],
    [0.5, 0.7],
    [-0.1, 0.2]
]

To retrieve the vector for Document 3, we can simply access embeddings[2], which gives us [0.8, -0.9].

#### Closing Summary
and Common Mistakes
Correctness: Ensure that each document has a unique identifier and is stored at the corresponding index.
Efficiency: Using an array ensures (O(1)) access time but requires enough memory to hold all vectors.
Mistakes:
Not ensuring uniqueness of document IDs can lead to overwriting or misidentification.
Failing to account for edge cases like out-of-bound indices.
Code Example

Here is a simple Python example:

PYTHON
# Sample vector embeddings for demonstration
embeddings = [
    [0.2, 0.3],
    [-0.4, 0.6],
    [0.8, -0.9],
    [0.5, 0.7],
    [-0.1, 0.2]
]

# Retrieving the i-th document's vector
def get_document_vector(index):
    return embeddings[index]

# Example usage
i = 3
document_vector = get_document_vector(i)
print(document_vector)  # Output: [0.5, 0.7]

**Code:**

```python
embeddings = [
    [0.2, 0.3],
    [-0.4, 0.6],
    [0.8, -0.9],
    [0.5, 0.7],
    [-0.1, 0.2]
]

def get_document_vector(index):
    return embeddings[index]

i = 3
document_vector = get_document_vector(i)
print(document_vector)  # Output: [0.5, 0.7]
```

**Example input / output:**

```text
embeddings = [
    [0.2, 0.3],
    [-0.4, 0.6],
    [0.8, -0.9],
    [0.5, 0.7],
    [-0.1, 0.2]
]

def get_document_vector(index):
    return embeddings[index]

i = 3
document_vector = get_document_vector(i)
print(document_vector)  # Output: [0.5, 0.7]
```

