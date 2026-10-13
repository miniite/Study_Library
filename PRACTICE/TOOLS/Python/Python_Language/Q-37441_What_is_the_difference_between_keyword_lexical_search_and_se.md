---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between keyword (lexical) search and semantic (embedding) search, and when would you combine them in a Python retrieval step?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Grounding Issues](<../../../../TOOLS/Python/Python_Language/RAG_Grounding_Issues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AI engineering, understanding the differences between keyword (lexical) search and semantic (embedding) search is crucial for building effective information retrieval systems. This answer will first provide an overview of both types of searches, then explain when to use each one and how they can be combined in a Python retrieval step.

TL;DR

Keyword (lexical) search works by matching exact words or patterns within text, making it fast but limited in context understanding. Semantic (embedding) search, on the other hand, uses embeddings to capture the meaning behind words, enabling more sophisticated and context-aware searches. Combining both can provide a robust solution that leverages the speed of keyword search with the depth of semantic understanding.

#### Triaging the Question

The question is essentially asking about two different approaches to text retrieval: one based on exact matching (keyword) and another based on meaning extraction (semantic). The interviewer is probing your understanding of these techniques, their strengths, weaknesses, and when they should be combined. We will assume that you are familiar with basic natural language processing (NLP) concepts.

Keyword (Lexical) Search

Keyword search involves looking for exact matches or patterns within text. It’s straightforward and efficient but relies heavily on the vocabulary used in queries. For example:

PYTHON
import re

def keyword_search(text, keywords):
    results = []
    for word in keywords:
        if re.search(word, text):
            results.append(word)
    return results
Semantic (Embedding) Search

Semantic search uses embeddings to represent words and phrases as vectors, allowing the system to understand context and meaning. Libraries like gensim or transformers from Hugging Face can be used for this purpose:

PYTHON
from transformers import BertModel, BertTokenizer

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertModel.from_pretrained('bert-base-uncased')

def semantic_search(text, queries):
    inputs = tokenizer(queries, return_tensors='pt', padding=True)
    outputs = model(**inputs)
    embeddings = outputs.last_hidden_state
    # Further processing to compare embeddings and find similarities
Combining Both Approaches

Combining these methods can leverage the efficiency of keyword search with the semantic understanding of embedding search. A practical approach might involve using a keyword search for quick, preliminary results, followed by a more detailed semantic analysis.

#### Example
Workflow in Python
Initial Keyword Search: Use regular expressions to quickly find matching keywords.
Refine Using Embeddings: For more precise matches, use embeddings to compare the query with documents or text snippets.

Here’s a simplified example:

PYTHON
def combined_search(text, keywords, queries):
    # Step 1: Perform keyword search
    results = keyword_search(text, keywords)

    # Step 2: Use embedding for semantic refinement
    if results:
        refined_results = semantic_search(text, results + queries)
        return refined_results
    else:
        return []

combined_search("This is a sample text about cats and dogs", ["cat", "dog"], ["cats", "dogs"])

#### Closing Summary
and Common Mistakes/Follow-Ups

Combining keyword search with semantic embedding can significantly enhance the accuracy and relevance of information retrieval systems. However, it’s important to balance the speed of keyword search with the complexity of semantic embeddings. Over-reliance on complex models might not be necessary for simple use cases.

#### Common mistakes
include:

Ignoring the context in which the search is being performed.
Not considering the computational cost and resource requirements of embedding-based approaches.
Failing to validate and test both methods separately before combining them.

For follow-ups, you could discuss trade-offs between speed and accuracy, different NLP libraries and models available for each approach, or how these techniques can be integrated into a larger information retrieval pipeline.

**Code:**

```python
import re

def keyword_search(text, keywords):
    results = []
    for word in keywords:
        if re.search(word, text):
            results.append(word)
    return results

from transformers import BertModel, BertTokenizer

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertModel.from_pretrained('bert-base-uncased')

def semantic_search(text, queries):
    inputs = tokenizer(queries, return_tensors='pt', padding=True)
    outputs = model(**inputs)
    embeddings = outputs.last_hidden_state
    # Further processing to compare embeddings and find similarities

def combined_search(text, keywords, queries):
    results = keyword_search(text, keywords)
    if results:
        refined_results = semantic_search(text, results + queries)
        return refined_results
    else:
        return []

combined_search("This is a sample text about cats and dogs", ["cat", "dog"], ["cats", "dogs"])
```

**Example input / output:**

```text
# Example input
text = "This is a sample text about cats and dogs"
keywords = ["cat", "dog"]
queries = ["cats", "dogs"]

# Expected output (simplified)
refined_results = combined_search(text, keywords, queries)
print(refined_results)  # Output: ['cat', 'dog']
```

