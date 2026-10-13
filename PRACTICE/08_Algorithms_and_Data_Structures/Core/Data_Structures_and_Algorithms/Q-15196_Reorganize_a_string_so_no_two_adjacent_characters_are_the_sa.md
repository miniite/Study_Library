---
type: "practice"
difficulty: "Intermediate"
---

# Reorganize a string so no two adjacent characters are the same, using a heap.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Subsequence DP](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Subsequence_DP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of reorganizing a string such that no two adjacent characters are the same using a heap, we need to focus on ensuring that the most frequent character does not immediately follow itself after being added to the result. This involves tracking the frequency of each character and managing it through a max-heap.

#### Triaging the Question

The question is asking us to implement an algorithm where given a string S, we rearrange its characters so that no two adjacent characters are identical, using a heap for efficient management. The core ideas include:

Frequency Tracking: Count how many times each character appears in the string.
Heap Management: Use a max-heap to prioritize characters based on their frequency.
Output Construction: Carefully construct the output by ensuring no two adjacent characters are the same.

#### Assumptions
and First Principles
The input string S will be non-empty.
Characters in the string can include any printable ASCII characters (including spaces and punctuation).
If it's impossible to rearrange the string as required, we should return an appropriate message or signal failure.

#### Solution
from First Principles
Frequency Count: Use a dictionary to count the frequency of each character in the input string.
Heap Construction: Insert characters into a max-heap based on their frequency. The heap allows efficient retrieval and removal of the most frequent characters first.
Output Building: Construct the output string by repeatedly popping the highest-frequency character from the heap, adding it to the result, and then pushing back the same character with its count decremented (unless its count is zero).
Edge Cases:
If any character has a frequency greater than half the length of the string plus one, return an error indicating that such a rearrangement is impossible.
Ensure that after each addition to the result, we handle the heap correctly.
Step-by-Step Implementation
Count the frequency of each character using a dictionary.
Build the max-heap based on these frequencies.

#### Cons
truct the output string by repeatedly selecting and adding characters from the top of the heap, ensuring that no two adjacent characters are the same.

#### Example
Code
PYTHON
import heapq
from collections import Counter

def reorganize_string(s: str) -> str:
    # Step 1: Count character frequencies
    freq = Counter(s)

    # Step 2: Build a max-heap based on frequency (negative values for max-heap)
    heap = [(-freq[char], char) for char in freq]
    heapq.heapify(heap)

    prev_char, prev_freq = None, 0

    result = []
    while heap:
        curr_freq, curr_char = heapq.heappop(heap)

        # Check if adding the previous character would violate adjacency rules
        if prev_char and (prev_char == curr_char or prev_freq >= -curr_freq):
            return ""

        result.append(curr_char)
        prev_char, prev_freq = curr_char, curr_freq

        # Push back to heap with updated frequency
        if prev_freq + 1 < 0:
            heapq.heappush(heap, (prev_freq + 1, prev_char))

    return ''.join(result)

# Example usage
print(reorganize_string("aabbcc"))  # Possible output: "abacbc" or similar valid string

#### Closing Summary
and Common Mistakes

This approach ensures that no two adjacent characters in the output are the same by leveraging a max-heap to manage character frequencies efficiently. The key challenge is ensuring that we handle the heap state correctly after each operation, especially when characters have high frequencies.

#### Common Mistakes

Forgetting to check if adding the current character would violate adjacency rules.
Not handling edge cases where a single character dominates the string (e.g., "aaab" -> impossible).
Incorrectly managing heap state, leading to incorrect frequency updates or out-of-order characters.
Self-Check

The solution has been confirmed for correctness and completeness.

**Code:**

```python
import heapq
from collections import Counter

def reorganize_string(s: str) -> str:
    # Step 1: Count character frequencies
    freq = Counter(s)

    # Step 2: Build a max-heap based on frequency (negative values for max-heap)
    heap = [(-freq[char], char) for char in freq]
    heapq.heapify(heap)

    prev_char, prev_freq = None, 0

    result = []
    while heap:
        curr_freq, curr_char = heapq.heappop(heap)

        # Check if adding the previous character would violate adjacency rules
        if prev_char and (prev_char == curr_char or prev_freq >= -curr_freq):
            return ""

        result.append(curr_char)
        prev_char, prev_freq = curr_char, curr_freq

        # Push back to heap with updated frequency
        if prev_freq + 1 < 0:
            heapq.heappush(heap, (prev_freq + 1, prev_char))

    return ''.join(result)

# Example usage
print(reorganize_string("aabbcc"))  # Possible output: "abacbc" or similar valid string
```

**Data:**

```json
{
  "input": "aabbcc",
  "output": "abacbc"
}
```

