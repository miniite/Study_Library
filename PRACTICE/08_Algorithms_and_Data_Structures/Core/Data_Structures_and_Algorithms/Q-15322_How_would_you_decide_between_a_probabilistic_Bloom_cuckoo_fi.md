---
type: "practice"
difficulty: "Deep"
---

# How would you decide between a probabilistic Bloom/cuckoo filter and an exact index for a 'have we sent this email before' check, and how do you reason about the cost of a false positive here?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Heaps and Running Median](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heaps_and_Running_Median.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to use a probabilistic Bloom or cuckoo filter versus an exact index for checking if "have we sent this email before," we need to carefully weigh several factors, including space usage, false positive rates, and query performance. Let's break down the key aspects from first principles.

#### Triaging the Question

The core of the question is about understanding the trade-offs between probabilistic data structures (Bloom filters and cuckoo filters) versus exact indexing in terms of space efficiency and accuracy. The interviewer will be probing your ability to reason through these trade-offs, your understanding of false positives, and how you would measure and balance these factors.

#### Assumptions
Space Constraints: Are there strict limits on storage?
False Positive Tolerance: How important is it to avoid false negatives (i.e., ensuring we never send a duplicate email by mistake)?
Query Frequency: How often will this check be performed?

#### First-Principles Solution
Bloom Filter

A Bloom filter uses a bit array and multiple hash functions to store elements. It can efficiently test whether an element is in the set, but it has a non-zero probability of false positives (i.e., saying an item was added when it wasn't). False negatives are impossible with a correctly implemented Bloom filter.

Cuckoo Filter

A cuckoo filter is a more advanced probabilistic data structure that combines elements from Bloom filters and cuckoo hashing. It can also test whether an element is in the set, but it offers better false positive rates and lower space usage compared to traditional Bloom filters. Moreover, it can handle deletions efficiently.

Exact Index

An exact index involves storing each email uniquely in a database or hash table, ensuring no duplicates. This approach guarantees zero false positives but consumes more storage.

Step-by-Step Reasoning

Space Efficiency:

Bloom filters use significantly less space than exact indices. For example, a 32-bit Bloom filter can store around 40 elements per bit if the desired false positive rate is low.
Cuckoo filters further reduce this footprint and offer better performance under high load.

False Positive Rate:

The false positive rate of a Bloom filter depends on its parameters (size, number of hash functions) and increases as more elements are added.
Cuckoo filters generally have lower false positive rates compared to Bloom filters for the same size and load factor.

Query Performance:

Both Bloom filters and cuckoo filters offer constant-time lookups, making them very efficient.
Exact indices also perform well but require more memory, which can impact performance in systems with strict storage constraints.

Scalability and Maintenance:

Bloom and cuckoo filters are easier to scale because they don't store the actual data. They just keep track of presence or absence using hashes.
Exact indices might need more complex maintenance if you have frequent deletions (though cuckoo filters handle this better).

#### Conclusion

Given these considerations, here’s how we would decide:

If space is severely limited and false positives can be tolerated to some degree, a Bloom filter or cuckoo filter would be the best choice.
For mission-critical systems where zero false negatives are mandatory (e.g., email sending), an exact index should be used.

#### Common Mistakes
and Follow-Ups
Misunderstanding False Positives: Some may overlook the importance of ensuring no false negatives in critical applications.
Ignoring Query Frequency: The frequency of checks might affect the choice; more frequent queries could tip the balance towards Bloom filters for lower overhead.
Overlooking Deletion Support: Cuckoo filters handle deletions better, which is a significant advantage over Bloom filters.

In summary, the decision hinges on balancing storage constraints with acceptable error rates and performance requirements.

