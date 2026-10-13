---
type: "practice"
difficulty: "Deep"
---

# You must decide between a probabilistic filter and an exact index for 'have we emailed this address before'; how do you reason about the business cost of a false positive versus the memory of an exact set?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Heaps and Running Median](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heaps_and_Running_Median.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between a probabilistic filter (like Bloom Filter) and an exact index for determining if we have emailed an address before, it's crucial to balance the business cost of false positives with the memory overhead of maintaining an exact set. In this scenario, our primary concerns are accuracy and space efficiency.

#### Triaging the Question

The core question is about evaluating the trade-offs between false positive rates and memory usage in a probabilistic filter versus using an exact set index. The interviewer likely wants to explore your understanding of these concepts and how you would apply them to make an informed decision based on business needs.

#### Assumptions
False Positive Cost: A false positive might lead to unnecessary retries or sending emails unnecessarily, which could incur costs in terms of email credits, bandwidth, or user experience.
Memory Usage: Using exact sets (like hash maps) can consume significant memory resources, especially for large datasets.

#### First-Principles Solution

False Positive Rate:

A Bloom Filter offers a false positive rate that is configurable and relatively low. This means the probability of incorrectly identifying an email address as having been sent to before is minimal but not zero.
An exact set (e.g., a hash map) has no false positives, so we can be certain if it exists or not.

Memory Usage:

A Bloom Filter uses significantly less memory compared to maintaining an exact set, making it more scalable for large datasets.
An exact set requires storing every email address in memory, which can become very expensive and impractical as the dataset grows.

Error Costs:

If a false positive occurs with a Bloom Filter, we might retry sending an email unnecessarily, but this cost is usually lower than the cost of not detecting an existing record (a false negative).
If we use an exact set, there are no false positives, so every check will be precise. However, the cost of maintaining such a set can be prohibitive.
Reasoning Step-by-Step

Define Costs:

Calculate the cost of sending unnecessary emails due to false positives with Bloom Filter. [ \text{Cost}{\text{FP}} = P(\text{FP}) \times C{\text{email}} ] where (P(\text{FP})) is the probability of a false positive and (C_{\text{email}}) is the cost per email.
Calculate the cost of maintaining an exact set index: [ \text{Cost}{\text{exact}} = S \times C{\text{memory}} ] where (S) is the number of unique email addresses and (C_{\text{memory}}) is the memory cost per address.

Compare Costs:

Determine which cost is more critical for your specific use case.

#### Cons
ider the business impact: If sending unnecessary emails is a high-cost operation, you might prefer Bloom Filter to avoid these costs. Conversely, if maintaining an exact set index is prohibitively expensive in terms of memory and it's acceptable to occasionally send duplicate emails, using Bloom Filter can be more cost-effective.

Choose Based on Business Requirements:

If the business can tolerate a small number of false positives, go with Bloom Filter.
If memory constraints are critical and you cannot afford the overhead of an exact set index, use Bloom Filter.
If precision is paramount and even one missed email could cause significant issues, invest in maintaining an exact set index.

#### Closing Summary

To summarize, deciding between a probabilistic filter and an exact index involves understanding the business costs associated with false positives versus memory usage. By quantifying these costs and aligning them with specific requirements, you can make an informed decision that best fits your use case.

#### Common Mistakes

Overlooking the potential impact of false positives on user experience.
Underestimating the long-term cost of maintaining a large exact set index.

#### Follow-Ups

How would you monitor the performance and costs associated with each approach?
What other factors might influence this decision, such as data growth rate or regulatory constraints?

**Comparison:**

```
Method	False Positives	Memory Usage
Bloom Filter	Low (configurable)	Low
Exact Set	None	High
```

**Code:**

```python
# Example code for Bloom Filter implementation
from bitarray import bitarray

class BloomFilter:
    def __init__(self, size=1024):
        self.size = size
        self.bit_array = bitarray(size)
        self.bit_array.setall(0)

    def add(self, item):
        hashes = [hash(item) % self.size,
                  hash(item + 'A') % self.size]
        for i in hashes:
            self.bit_array[i] = True

    def query(self, item):
        hashes = [hash(item) % self.size,
                  hash(item + 'A') % self.size]
        if all(self.bit_array[i] for i in hashes):
            return True
        else:
            return False
```

**Example input / output:**

```text
Given the input of an email address, a Bloom Filter would be queried as follows:

- `filter = BloomFilter()`
- `for email in emails: filter.add(email['address'])`
- To check if an address has been sent to before: `filter.query('user1@example.com')` returns `True`.
```

**Data:**

```json
{
  "emails": [
    {"address": "user1@example.com"},
    {"address": "user2@example.com"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Define Costs
4. Are false positives costly?
5. Use Bloom Filter
6. Maintain Exact Set Index

