---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute the average embedding of a set of chunk vectors to get a document-level vector, and what is the time and space complexity?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Similarity Search Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Similarity_Search_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the average embedding of a set of chunk vectors to get a document-level vector, we need to sum up all the individual vectors and then divide by the total number of vectors. This process can be understood in terms of both time and space complexity.

#### Triaging the Question

The question is asking for an efficient way to compute the average embedding of multiple vectors. The interviewer is likely probing your understanding of vector arithmetic, data structures, and algorithmic optimization. We'll assume that each chunk vector has ( n ) dimensions, and there are ( m ) such vectors.

Step-by-Step Solution
1. Summing Vectors

To compute the average embedding, we first need to sum all the chunk vectors. Let's denote the set of chunk vectors as ( {v_1, v_2, ..., v_m} ), where each vector ( v_i ) is an ( n )-dimensional vector.

PYTHON
def sum_vectors(vectors):
    # Initialize a zero vector to accumulate the sum
    total = [0] * len(vectors[0])

    for vector in vectors:
        for i in range(len(vector)):
            total[i] += vector[i]

    return total
2. Dividing by Number of Vectors

Once we have the summed vector, we divide each dimension by ( m ) to get the average.

PYTHON
def compute_average_embedding(vectors):
    # Sum all vectors
    sum_vector = sum_vectors(vectors)

    # Compute the average embedding
    average_embedding = [x / len(vectors) for x in sum_vector]

    return average_embedding
Time and Space Complexity

Time Complexity: The time complexity of this approach is ( O(m \cdot n) ). This is because we need to iterate over all vectors (of length ( m )) and their dimensions (length ( n )).

Space Complexity: The space complexity is ( O(n) ) for storing the sum vector, as each vector has ( n ) dimensions.

#### Closing Summary

The process of computing the average embedding involves summing up all chunk vectors and then dividing by the number of vectors. This approach ensures that we capture the collective information of multiple chunks to form a document-level vector representation. Understanding the time and space complexities helps in optimizing performance, especially when dealing with large datasets.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to initialize the sum vector correctly or not handling edge cases where ( m = 0 ).

#### Follow-Up
Discussing how to handle large vectors (e.g., using sparse representations) or parallelizing the computation for efficiency.

