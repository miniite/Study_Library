---
type: "practice"
difficulty: "Basic"
---

# What data structure would you use to map a model name to its per-token pricing so you can estimate an LLM call's cost, and what is the lookup complexity?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Top-k and Sketches](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Top-k_and_Sketches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of mapping a model name to its per-token pricing for estimating an LLM (Large Language Model) call's cost, we need to choose an appropriate data structure that allows efficient lookups and updates. The key requirement here is fast lookup based on the model name. Given this context, a hash table or dictionary would be the most suitable choice.

#### Triaging the Question

The question essentially asks about selecting a data structure for storing mappings between strings (model names) and numerical values (per-token prices), with an emphasis on efficient lookup operations. The interviewer is likely probing your understanding of basic data structures like dictionaries, hash tables, and their associated performance characteristics.

#### Assumptions
The model names are unique.
We need to frequently look up the per-token price based on the model name.
There might be a need for occasional updates or insertions of new mappings.

#### First-Principles Solution

A hash table (also known as a dictionary in Python) is an ideal choice for this problem due to its constant time complexity for lookups, which is (O(1)). Here’s why:

Mapping Model Name to Price: We can use the model name as the key and the per-token price as the value.
Efficient Lookups: Given a model name, we can quickly determine its corresponding per-token price using hash table lookups.

Let's consider an example in Python:

PYTHON
# Example implementation
model_prices = {
    'gpt-3': 0.002,
    'gpt-4': 0.0015,
    'text-davinci-003': 0.0016
}

# Lookup operation
print(model_prices['gpt-3'])  # Output: 0.002

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, using a hash table ensures that lookups are very fast, making it an excellent choice for estimating the cost of LLM calls based on model names. The primary complexity is (O(1)), which means each lookup operation will be quick regardless of the number of entries in the dictionary.

#### Common mistakes
to avoid:

Using a list or array, where lookups would be (O(n)) and not as efficient.
Ignoring the need for fast updates and inserts if necessary; hash tables support these operations with average complexity (O(1)).

If asked follow-ups, you might be asked about handling collisions in hash tables (e.g., chaining or open addressing), which are advanced topics but can be discussed briefly if relevant.

**Code:**

```python
model_prices = {
    'gpt-3': 0.002,
    'gpt-4': 0.0015,
    'text-davinci-003': 0.0016
}

# Lookup operation
print(model_prices['gpt-3'])  # Output: 0.002
```

**Example input / output:**

```

Input:

PYTHON
Copy
model_prices = {
    'gpt-3': 0.002,
    'gpt-4': 0.0015,
    'text-davinci-003': 0.0016
}

print(model_prices['gpt-3'])

Output:

PLAINTEXT
Copy
0.002
```

