---
type: "practice"
difficulty: "Intermediate"
---

# How would you store few-shot examples so you can quickly pick the most relevant ones for a prompt, and what structure supports similarity-based selection?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Arrays and Embedding Lookup](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays_and_Embedding_Lookup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to design a data structure that efficiently stores few-shot examples and allows us to select the most relevant ones based on similarity to a given prompt. The key is to ensure quick retrieval and comparison operations. For an intermediate-level AI engineer, we'll focus on using hash tables for fast lookups and leveraging similarity metrics like cosine similarity.

#### Triaging the Question

The question asks about designing a data structure to store few-shot examples efficiently and select relevant ones based on their similarity to a prompt. The main aspects are:

Data Storage: How to represent and store the examples.
Similarity Calculation: What metric will be used to determine relevance.
Efficient Retrieval: How to quickly retrieve the most relevant examples.

#### First Principles Solution
Representation of Examples: Each example is represented as a vector in a high-dimensional space, where each dimension corresponds to a feature or token.
Hash Table for Fast Lookup: Use a hash table (dictionary) to store these vectors by their index or unique identifier.
Similarity Metric: Choose a similarity metric like cosine similarity to compare the example vectors with the query vector.
Retrieval Algorithm:
Compute the query vector from the prompt.
Calculate the cosine similarity between the query vector and each stored example vector.
Sort the examples based on their similarity scores and return the top-k most relevant examples.
Detailed Solution
Vector Representation: Each few-shot example is converted into a high-dimensional feature vector using techniques like TF-IDF, word embeddings (e.g., Word2Vec or BERT embeddings), or any other suitable method depending on the nature of the data.
Hash Table Setup:
PYTHON

#### example
s = {
    "example1_id": example1_vector,
    "example2_id": example2_vector,
    ...
}
Cosine Similarity Calculation: To compare a query vector with an example vector, use the cosine similarity formula: [ \text{similarity} = \frac{\mathbf{A} \cdot \mathbf{B}}{|\mathbf{A}| |\mathbf{B}|} ]
Retrieval Algorithm:
Compute query vector from the prompt.
Iterate over all example vectors, calculate cosine similarity for each, and store the results in a sorted list.
Return the top-k examples with highest similarities.

#### Closing Summary

The chosen data structure (hash table) ensures fast insertion, deletion, and lookup operations. Cosine similarity provides a robust way to measure vector-based similarity, making it suitable for text or feature vectors. This approach balances efficiency and accuracy in selecting relevant few-shot examples based on the query prompt.

#### Common Mistakes/Follow-Ups
Using an Inefficient Similarity Metric: Using metrics like Euclidean distance can be computationally expensive; cosine similarity is preferable.
Ignoring Vector Normalization: Ensure all vectors are normalized before computing similarities to avoid biases due to vector magnitudes.
Choosing k Too Large or Small: Selecting an appropriate (k) value ensures that the number of returned examples is manageable and relevant.

