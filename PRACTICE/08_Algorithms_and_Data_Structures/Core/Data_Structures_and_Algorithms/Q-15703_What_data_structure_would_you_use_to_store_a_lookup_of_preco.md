---
type: "practice"
difficulty: "Basic"
---

# What data structure would you use to store a lookup of precomputed embeddings for frequently asked questions so you skip re-embedding them, and why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Arrays and Embedding Lookup](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays_and_Embedding_Lookup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently store precomputed embeddings for frequently asked questions (FAQs), an appropriate data structure is a hash table or dictionary. This choice leverages constant-time complexity for lookups, making it ideal for scenarios where you need to quickly retrieve preprocessed embeddings without the overhead of recalculating them.

The question essentially asks why we should use a hash table over other possible structures such as arrays, linked lists, or trees. Let's delve into the reasoning from first principles.

**Code:**

```python
# Example Python implementation using a dictionary to store precomputed embeddings
faq_embeddings = {
    "What is AI?": [0.1, 0.2, -0.3],
    "How does it work?": [-0.4, 0.5, 0.6],
    # More FAQs and their corresponding embeddings...
}
```

**Data:**

```

The data schema for storing precomputed embeddings in a dictionary might look like:

YAML
Copy
faq_embeddings:
  question1: [embedding1_value1, embedding1_value2, ...]
  question2: [embedding2_value1, embedding2_value2, ...]
FAQ	Embedding Vector
What is AI?	[0.1, 0.2, -0.3]
How does it work?	[-0.4, 0.5, 0.6]
```

