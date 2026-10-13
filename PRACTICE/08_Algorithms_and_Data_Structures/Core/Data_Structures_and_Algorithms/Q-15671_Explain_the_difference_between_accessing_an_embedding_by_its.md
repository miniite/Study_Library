---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between accessing an embedding by its integer index in an array versus by a string ID in a dict, in terms of complexity and use case.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Arrays and Embedding Lookup](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays_and_Embedding_Lookup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between accessing an embedding by its integer index in an array versus by a string ID in a dictionary, let's break down the concepts first. Essentially, both methods are used to retrieve embeddings (vectors representing words or objects) from a data structure. However, they differ significantly in terms of time complexity and use cases.

When you access an embedding by its integer index in an array, it is straightforward and efficient. An array provides constant-time O(1) access, meaning the time taken to access an element does not depend on the size of the array. This makes it ideal for scenarios where quick lookups are essential.

On the other hand, accessing an embedding via a string ID using a dictionary involves additional steps due to the hash function and collision resolution mechanisms. Dictionaries provide average O(1) time complexity but can degrade to O(n) in worst-case scenarios, especially when there are many collisions.

In summary, arrays are faster for direct index access but require pre-allocating space or handling out-of-bounds errors if you do not know the exact index range. Dictionaries offer more flexibility with string keys and handle dynamic insertion, deletion, and updates seamlessly.

**Code:**

```python
# Example of accessing embeddings by integer index in an array
embedding_array = [10, 20, 30, 40]  # Predefined embeddings as integers

def get_embedding(index):
    return embedding_array[index]

print(get_embedding(2))  # Output: 30

# Example of accessing embeddings by string ID in a dictionary
embeddings_dict = {"apple": [10, 20, 30], "banana": [40, 50, 60]}

def get_embedding_dict(key):
    return embeddings_dict[key]

print(get_embedding_dict("apple"))  # Output: [10, 20, 30]
```

**Example input / output:**

```text
Accessing an embedding by index in array:
- Input: `get_embedding(2)`
- Output: `[30]`

Accessing an embedding by string ID in dictionary:
- Input: `get_embedding_dict("apple")`
- Output: `[10, 20, 30]`
```

**Data:**

```json
{
  "embeddings_array": [10, 20, 30, 40],
  "embeddings_dict": {
    "apple": [10, 20, 30],
    "banana": [40, 50, 60]
  }
}
```

