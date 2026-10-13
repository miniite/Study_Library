---
type: "practice"
difficulty: "Basic"
---

# Why does counting tokens to stay under an LLM context window require iterating the token list, and what is the time complexity of that count?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Prefix Sums and Sliding Windows](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sums_and_Sliding_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Large Language Models (LLMs), staying within their context windows is crucial because it ensures you can make efficient use of the model's capabilities without hitting rate limits or exceeding token capacity constraints. The process of counting tokens to ensure we stay under a specific LLM context window involves iterating through each token in the text, which provides us with precise control over how much input data we can send.

Essentially, we need to iterate through the list of tokens because:

Each token might have a different size (number of bytes or characters), and LLMs often handle these differently.
The exact number of tokens must be counted accurately for models like GPT-3 or other transformer-based architectures that use tokenization.

The time complexity of this operation is O(n), where n represents the number of tokens in the input text. This linear time complexity arises because we need to visit each token exactly once to count it, and there's no faster way to do this without prior knowledge of the total length or structure of the input.

To understand why iterating through the list is necessary:

Tokenization Variability: LLMs tokenize input text differently. For example, spaces, punctuation, and special characters might be treated as individual tokens. Thus, counting each token ensures we accurately measure against the context window.
Dynamic Token Lengths: In some cases, a single character could expand to multiple tokens (e.g., emojis or complex Unicode symbols), so simply measuring by length won't suffice.

Here's an example of how you might implement this in Python:

PYTHON
def count_tokens(text):
    # Assume we have a tokenizer function that splits the text into tokens.
    from some_tokenizer_library import tokenize

    tokens = tokenize(text)
    return len(tokens)

# Example usage
text = "Hello, world! This is an example sentence."
token_count = count_tokens(text)
print(f"Token count: {token_count}")

**Code:**

```python
def count_tokens(text):
    from some_tokenizer_library import tokenize

    tokens = tokenize(text)
    return len(tokens)

# Example usage
text = "Hello, world! This is an example sentence."
token_count = count_tokens(text)
print(f"Token count: {token_count}")
```

**Example input / output:**

```text
Input text: "Hello, world! This is an example sentence."
Output token count: 8
```

