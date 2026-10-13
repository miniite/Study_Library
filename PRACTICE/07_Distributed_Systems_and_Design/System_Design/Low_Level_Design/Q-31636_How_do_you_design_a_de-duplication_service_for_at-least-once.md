---
type: "practice"
difficulty: "Deep"
---

# How do you design a de-duplication service for at-least-once events so the state doesn't grow unbounded, and what's the trade-off between a Bloom filter, a TTL set, and a database unique constraint?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumer Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a de-duplication service for at-least-once events that ensures state does not grow unbounded, we need to consider the trade-offs between using a Bloom filter, maintaining an expiry time (TTL) set, and leveraging a database unique constraint. The core challenge is balancing storage efficiency with correctness guarantees.

#### Triaging the Question

The interviewer is probing our understanding of low-level design principles, especially in managing state without overgrowth. They are also interested in our ability to evaluate different data structures and their trade-offs in terms of space complexity, time complexity, and correctness.

#### Assumptions
Event Volume: The service will handle a large volume of events.
Event Uniqueness: Events can be uniquely identified by a key (e.g., event ID).
Storage Constraints: We must ensure that storage does not grow unbounded over time.

#### First-Principles Solution
Bloom Filter

A Bloom filter is an efficient, probabilistic data structure used to test whether an element is a member of a set. It provides a quick way to check if an event has been seen before with minimal space overhead but at the cost of false positives.

#### Pros

High storage efficiency (small footprint).
Fast lookup operations.
Can handle large datasets.

#### Cons

False positive rate, which might lead to incorrect deduplication in critical scenarios.
No way to remove entries; once added, they stay.
TTL Set

A TTL set uses a more deterministic approach by associating each event with an expiry time. Events are kept for their lifetime and removed after the TTL expires. This ensures that state does not grow indefinitely but can lead to unnecessary storage if events have long lifetimes or frequent updates.

#### Pros

Ensures bounded growth of state.
Correctness is guaranteed as expired entries will be removed.

#### Cons

Higher space overhead compared to Bloom filters.
More complex implementation and maintenance due to managing TTLs.
Potential for data inconsistencies if the TTL logic gets complicated.
Database Unique Constraint

A database unique constraint enforces uniqueness at the database level, ensuring that an event can only be stored once. This approach leverages ACID properties but requires a robust database system with overhead like indexing and transaction management.

#### Pros

Ensures absolute correctness.
Leverages well-tested database mechanisms for consistency.

#### Cons

Higher storage requirements due to additional indexes.
Potential performance impact from write operations on the database.
Complexity in managing distributed transactions if using a relational database.

#### Trade-offs

The choice between these approaches depends heavily on the specific use case and constraints. Here are some key factors:

Space vs. Correctness: Bloom filters offer minimal space usage but risk false positives, while a TTL set or unique constraint ensures correctness at the cost of higher storage.
Scalability and Performance: Bloom filters provide fast lookup times with low overhead, making them ideal for high-throughput systems. TTL sets are more suitable when state must be managed precisely over time.
Fault Tolerance and Consistency: Unique constraints ensure strong consistency but may introduce bottlenecks in distributed systems.

#### Closing Summary

In designing a de-duplication service, the trade-offs between Bloom filters, TTL sets, and database unique constraints revolve around storage efficiency, correctness guarantees, and system complexity. Each approach has its strengths and weaknesses, making the choice dependent on the specific requirements of the application.

#### Common Mistakes/Follow-Ups
Overlooking false positives in Bloom filters: Failing to account for the probabilistic nature can lead to incorrect deduplication.
Ignoring database overhead: Not considering the performance implications of unique constraints and index management.
Simplifying TTL logic: Overlooking edge cases where events may need more flexible handling than a fixed TTL.

Would you like me to elaborate on any specific trade-off or provide a code example for implementing one of these approaches?

**Comparison:**

```
Data Structure	Space Efficiency	Correctness	Complexity
Bloom Filter	High	Probabilistic	Low
TTL Set	Medium	Deterministic	Moderate
Database Unique Constraint	Low	Absolute	High
```

**Code:**

```python
# Example implementation of Bloom Filter in Python

class BloomFilter:
    def __init__(self, bit_size=1024):
        self.bit_array = [False] * bit_size
        self.false_positive_rate = 0.01  # Adjust based on requirements

    def add(self, key):
        for index in self.get_hash_indices(key):
            self.bit_array[index] = True

    def check(self, key):
        return all([self.bit_array[index] for index in self.get_hash_indices(key)])

    def get_hash_indices(self, key):
        # Simple hash function
        hashes = []
        seed = 131  # A prime number less than 256
        current_value = hash(key)
        while len(hashes) < 4:
            current_value = (current_value * seed) % len(self.bit_array)
            if current_value not in hashes:
                hashes.append(current_value)
        return hashes
```

**Example input / output:**

```text
bloom_filter = BloomFilter()
bloom_filter.add("event_id_12345")
print(bloom_filter.check("event_id_12345"))  # True
print(bloom_filter.check("nonexistent_event"))  # False (or True with false positive)
```

**Data:**

```json
{
  "event_id": "12345",
  "timestamp": "2023-10-01T00:00:00Z"
}
```

**Flow:**

1. True
2. False
3. True
4. False
5. True
6. False
7. Check Event
8. Bloom Filter
9. Add Event
10. TTL Set Check
11. Deduplicate and Add to TTLSet
12. Database Unique ConstraintCheck
13. Deduplicate and Add toDatabase
14. Log Error

