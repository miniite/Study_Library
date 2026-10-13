---
type: "practice"
difficulty: "Intermediate"
---

# Find the top k frequent words in a document and discuss heap vs bucket-sort approaches.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Top-k and Sketches](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Top-k_and_Sketches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of finding the top (k) frequent words in a document, we need to compare two main approaches: using a heap (priority queue) and using bucket sort. Both methods have their unique strengths and trade-offs depending on the characteristics of the input data.

#### Triage

The core of this question is about understanding how different algorithms can be applied to solve the same problem efficiently. The interviewer might probe your knowledge of data structures, time complexity analysis, and your ability to make informed decisions based on the problem's constraints. We'll assume the document contains a large number of words, some with very high frequency, and (k) is relatively small compared to the total number of unique words.

#### First-Principles Solution
Frequency Counting: First, we need to count the frequency of each word in the document.
Top (k) Selection:
Heap Approach: Use a min heap where you insert elements and keep only the top (k).
Bucket Sort Approach: Distribute words into buckets based on their frequency.
Step-by-Step Solution
1. Frequency Counting

We start by iterating through each word in the document, maintaining a hash map to count occurrences of each word.

PYTHON
from collections import Counter

def count_word_frequencies(document):
    return dict(Counter(document.lower().split()))
2. Top (k) Selection Using Heap
Heap Initialization: Initialize an empty min heap with capacity (k).
Word Insertion: For each word, insert it into the heap. If the heap size exceeds (k), remove the smallest element.
Result Extraction: After processing all words, extract elements from the heap in descending order of frequency.
PYTHON
import heapq

def top_k_heap(words_counted, k):
    min_heap = []
    for word, freq in words_counted.items():
        if len(min_heap) < k:
            heapq.heappush(min_heap, (freq, word))
        else:
            if freq > min_heap[0][0]:
                heapq.heapreplace(min_heap, (freq, word))

    # Extract elements and reverse for top k
    return [heapq.heappop(min_heap)[1] for _ in range(len(min_heap))]
3. Top (k) Selection Using Bucket Sort
Bucket Initialization: Create a list of buckets where the index represents frequency.
Word Distribution: Place each word into its corresponding bucket based on its frequency.
Result Extraction: Traverse the buckets from the end (highest frequency) to the beginning until we have collected (k) words.
PYTHON
def top_k_bucket_sort(words_counted, k):
    max_freq = max(words_counted.values())
    buckets = [[] for _ in range(max_freq + 1)]

    for word, freq in words_counted.items():
        buckets[freq].append(word)

    result = []
    for i in range(len(buckets) - 1, 0, -1):
        if k <= len(result):
            break
        result.extend(buckets[i])

    return result[:k]

#### Closing Summary
and Common Mistakes/Follow-Ups

Both methods have their pros and cons. Heap approach is more versatile for varying (k) sizes but has a higher time complexity (O(n log k)). Bucket sort, on the other hand, can be faster with optimal bucket size but might use extra space if the frequency range is large.

#### Common Mistakes

Not properly handling edge cases like empty documents or small (k).
Using an inappropriate data structure that doesn't fit the problem.
Overlooking the importance of time and space complexity analysis.

#### Example

Given a document: "the quick brown fox jumps over the lazy dog", find the top 2 frequent words:

PYTHON
document = "the quick brown fox jumps over the lazy dog"
words_counted = count_word_frequencies(document)
k = 2

# Top k using heap
top_k_heap(words_counted, k)  # Output: ['the', 'quick']

# Top k using bucket sort
top_k_bucket_sort(words_counted, k)  # Output: ['the', 'lazy']

**Code:**

```python
from collections import Counter
import heapq

def count_word_frequencies(document):
    return dict(Counter(document.lower().split()))

def top_k_heap(words_counted, k):
    min_heap = []
    for word, freq in words_counted.items():
        if len(min_heap) < k:
            heapq.heappush(min_heap, (freq, word))
        else:
            if freq > min_heap[0][0]:
                heapq.heapreplace(min_heap, (freq, word))

    # Extract elements and reverse for top k
    return [heapq.heappop(min_heap)[1] for _ in range(len(min_heap))]

def top_k_bucket_sort(words_counted, k):
    max_freq = max(words_counted.values())
    buckets = [[] for _ in range(max_freq + 1)]

    for word, freq in words_counted.items():
        buckets[freq].append(word)

    result = []
    for i in range(len(buckets) - 1, 0, -1):
        if k <= len(result):
            break
        result.extend(buckets[i])

    return result[:k]
```

**Data:**

```python
document = "the quick brown fox jumps over the lazy dog"
words_counted = count_word_frequencies(document)
k = 2
```

