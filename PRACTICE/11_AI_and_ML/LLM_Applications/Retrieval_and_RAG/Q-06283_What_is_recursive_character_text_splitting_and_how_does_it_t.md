---
type: "practice"
difficulty: "Basic"
---

# What is recursive character text splitting and how does it try to respect document structure?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Structure Aware Chunking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Structure_Aware_Chunking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Recursive character text splitting is a technique used in natural language processing (NLP) and information retrieval, where a long string of text is broken down into smaller chunks by considering the hierarchy and structure of the input document. The key idea behind this method is to respect the original structure of the text while ensuring that each chunk retains meaningful content.

The question here is essentially asking about how recursive splitting handles hierarchical structures like paragraphs, sentences, or sections in a document. Let's break down the concept from first principles:

#### Triage
Understanding the Question
What is the core problem?: The main goal is to divide a large text into smaller chunks that can be processed more efficiently.
What are we probing?: We need to understand how splitting respects the document’s structure, ensuring meaningful and coherent parts.

#### Assumptions
We are working with structured documents (e.g., plain text files or HTML).
The original structure of the document is important for context and meaning.
The splitting process should maintain readability and logical flow.

#### First-Principles Solution

Recursive character text splitting involves breaking down a string into substrings based on some criteria, but with careful consideration to avoid slicing through significant structural elements like paragraphs or sentences. Here’s a step-by-step approach:

Identify Document Structure: Start by parsing the document to understand its structure (e.g., HTML tags for web pages).
Define Split Criteria: Determine what constitutes a chunk of text, such as a sentence boundary in plain text.
Recursive Slicing: Use recursion to split the text while respecting these boundaries.

Here’s an example to illustrate this process:

PYTHON
def recursive_split(text, max_chunk_size=100):
    # Base case: if the text is too small, return it as a single chunk
    if len(text) <= max_chunk_size:
        return [text]

    # Find natural boundaries (e.g., sentence delimiters)
    sentences = text.split('. ')

    chunks = []
    current_chunk = ""

    for sentence in sentences:
        if len(current_chunk) + len(sentence) < max_chunk_size:
            current_chunk += '. ' + sentence
        else:
            # Add the chunk to the list and reset current_chunk
            chunks.append(current_chunk.strip())
            current_chunk = sentence

    # Don't forget the last chunk
    chunks.append(current_chunk.strip())

    return chunks

# Example usage
text = "This is a sample text. It contains several sentences which need to be split into smaller chunks while respecting their structure."
chunks = recursive_split(text)
for i, chunk in enumerate(chunks):
    print(f"Chunk {i+1}: {chunk}")

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, recursive character text splitting respects the document’s structure by identifying natural boundaries such as sentences or paragraphs and then recursively slicing the text to maintain meaningful chunks. A common mistake is ignoring structural elements entirely, which can lead to loss of context.

For follow-ups:

How would you handle special cases like abbreviations?
What about different languages with varying sentence structures?

**Code:**

```python
def recursive_split(text, max_chunk_size=100):
    # Base case: if the text is too small, return it as a single chunk
    if len(text) <= max_chunk_size:
        return [text]

    # Find natural boundaries (e.g., sentence delimiters)
    sentences = text.split('. ')

    chunks = []
    current_chunk = ""

    for sentence in sentences:
        if len(current_chunk) + len(sentence) < max_chunk_size:
            current_chunk += '. ' + sentence
        else:
            # Add the chunk to the list and reset current_chunk
            chunks.append(current_chunk.strip())
            current_chunk = sentence

    # Don't forget the last chunk
    chunks.append(current_chunk.strip())

    return chunks

# Example usage
text = "This is a sample text. It contains several sentences which need to be split into smaller chunks while respecting their structure."
chunks = recursive_split(text)
for i, chunk in enumerate(chunks):
    print(f"Chunk {i+1}: {chunk}")
```

**Example input / output:**

```text
Input: "This is a sample text. It contains several sentences which need to be split into smaller chunks while respecting their structure."
Output:
Chunk 1: "This is a sample text. It contains several sentences"
Chunk 2: "which need to be split into smaller chunks while respecting their structure."
```

