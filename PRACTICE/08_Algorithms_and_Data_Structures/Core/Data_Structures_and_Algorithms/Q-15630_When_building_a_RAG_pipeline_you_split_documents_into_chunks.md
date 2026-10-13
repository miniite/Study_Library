---
type: "practice"
difficulty: "Basic"
---

# When building a RAG pipeline you split documents into chunks; what data structure would you use to keep each chunk alongside its source metadata and why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Diff and LCS Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Diff_and_LCS_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently manage document chunks and their associated metadata in a RAG (Retrieval-Augmented Generation) pipeline, we typically need a data structure that can handle both the textual content of each chunk and its related metadata. The most suitable choice here is a dictionary or hash map, as it allows us to store key-value pairs where the keys are unique identifiers for each document chunk, and the values are objects containing the chunk text and its associated metadata.

To break down this solution further:

Understanding the Question: The interviewer is probing your understanding of how data structures can be used in practical scenarios, such as managing document chunks in a RAG pipeline.

#### Assumptions
We assume that each document chunk has unique identifiers (e.g., UUIDs) and metadata like the source document name, section, timestamp, etc.

#### First-Principles Solution
Choose the Data Structure: A dictionary or hash map is ideal because it provides fast lookups based on keys. This is crucial for operations such as adding new chunks, updating existing ones, and retrieving specific chunks quickly.
Metadata Handling: Each chunk's metadata will be stored in a nested structure (e.g., another dictionary), allowing us to easily access and manipulate the data.

Here’s how you might implement this in Python:

PYTHON
class DocumentChunk:
    def __init__(self, text, metadata):
        self.text = text
        self.metadata = metadata

# Example usage
chunks = {}
chunk_id = "1234567890"  # Unique identifier for the document chunk
metadata = {"source_document": "report.pdf", "section": "Introduction", "timestamp": "2023-10-01"}
chunk = DocumentChunk("This is a sample chunk text.", metadata)
chunks[chunk_id] = chunk

# Retrieving and updating chunks
print(chunks[chunk_id].text)  # Output: This is a sample chunk text.
chunks[chunk_id].metadata["section"] = "Conclusion"

#### Closing Summary
+ Common Mistakes/Follow-Ups: Using a dictionary ensures efficient storage and retrieval of document chunks with their metadata. However, consider the trade-offs like memory usage if you're dealing with a very large number of chunks. You might want to explore more advanced data structures or indexing strategies for scalability.

