---
type: "practice"
difficulty: "Basic"
---

# What is Chroma as a local vector store, and how would you create a collection and add documents to it in Python?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Query Results](<../../../../TOOLS/Python/Python_Language/Streaming_Query_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Chroma is an open-source vector database that allows you to efficiently store and query high-dimensional vectors, making it ideal for tasks like information retrieval or similarity searches. In this context, a collection within Chroma serves as a container for documents represented as vectors.

To create a collection in Python and add documents to it using Chroma, follow these steps:

#### Overview
First, you need to install the Chroma library.

#### Triage
The question asks about creating a collection and adding documents. We will use the  chromadb package to achieve this. You'll also need to understand how vectorization works, as each document is converted into a numerical vector before being stored.

#### Assumptions

You have Python installed on your system.
You are familiar with basic Python programming concepts and understand vectors in the context of machine learning.

#### First Principles Solution
Install the chromadb package using pip: pip install chromadb.
Import the necessary modules from the library.
Initialize a client to connect to your Chroma database.
Create a collection within the database.
Add documents and their corresponding vectors to the collection.

Here is a step-by-step implementation:

PYTHON
# Step 1: Install the chromadb package (if not already installed)
!pip install chromadb

# Step 2: Import necessary modules
from chromadb.client import Chroma

# Step 3: Initialize the client and create a collection
client = Chroma()
collection = client.create_collection(name="example_collection")

# Step 4: Define some sample documents and their vectors (for demonstration purposes)
documents = [
    "This is the first document.",
    "Here is another one!",
    "And yet another document to add to the collection."
]

vectors = [[0.1, 0.2, 0.3], [0.4, 0.5, 0.6], [0.7, 0.8, 0.9]]  # Random vectors for demonstration

# Step 5: Add documents and their corresponding vectors to the collection
collection.add(documents=documents, metadatas=[{"id": i} for i in range(len(documents))], embeddings=vectors)

#### Closing Summary
The above code snippet demonstrates how to create a Chroma collection and add documents with associated vectors. Ensure that you replace the sample data with your actual dataset.

#### Common Mistakes/Follow-Ups

Mistake 1: Not properly installing the chromadb package.
Mistake 2: Failing to initialize the client before creating a collection.
Mistake 3: Adding documents without vectors or vice versa.

#### Follow-up
How would you handle large-scale data? What about vector similarity queries?

**Code:**

```python
from chromadb.client import Chroma

client = Chroma()
collection = client.create_collection(name="example_collection")

documents = [
    "This is the first document.",
    "Here is another one!",
    "And yet another document to add to the collection."
]

vectors = [[0.1, 0.2, 0.3], [0.4, 0.5, 0.6], [0.7, 0.8, 0.9]]

collection.add(documents=documents, metadatas=[{"id": i} for i in range(len(documents))], embeddings=vectors)
```

