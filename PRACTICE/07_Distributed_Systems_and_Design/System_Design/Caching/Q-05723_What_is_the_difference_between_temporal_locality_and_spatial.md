---
type: "practice"
difficulty: "Basic"
---

# What is the difference between temporal locality and spatial locality?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Caching Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Temporal locality and spatial locality are two important concepts in caching that help us understand how data access patterns can be exploited to improve performance. In essence, temporal locality refers to the tendency of a program to reuse the same data or code over short intervals of time, while spatial locality means that if a particular piece of data is accessed, it's likely that nearby data will also be needed soon.

#### Triage
Restating the Question

The question is essentially asking us to explain the fundamental differences between these two caching concepts. Temporal locality deals with accessing the same data repeatedly in quick succession, whereas spatial locality involves accessing related or nearby data items together.

#### First-Principles Solution
Temporal Locality

Temporal locality can be understood by considering a simple example: when you're reading a book, you might read through several pages and then return to read the same page multiple times. This pattern of revisiting the same content is an example of temporal locality. In caching terms, if a particular piece of data is accessed frequently in quick succession (e.g., within milliseconds), it's likely that this data will be needed again soon.

Spatial Locality

On the other hand, spatial locality can be illustrated by thinking about how you navigate through a book. Once you start reading a chapter, it’s common to read several pages close to each other before moving on to another section of the book. Similarly, in computer systems, if a program accesses data from addresses that are close to each other in memory (e.g., consecutive lines in an array), then nearby data is likely to be accessed soon as well.

#### Closing Summary
and Common Mistakes/Follow-Ups

Temporal locality and spatial locality are crucial for understanding how caching works efficiently. By recognizing these patterns, cache algorithms can make more informed decisions on what data to keep in memory, thus improving overall performance. Common mistakes include misunderstanding the distinctions between temporal and spatial locality or ignoring one in favor of the other when designing a caching strategy.

#### Follow-up
questions might explore specific caching algorithms that leverage either or both types of locality, such as LRU (Least Recently Used) for temporal locality or LFU (Least Frequently Used) for spatial locality combined with temporal patterns.

**Comparison:**

| Temporal Locality | Spatial Locality |
|-------------------|------------------|
| Reusing the same data in quick succession. | Accessing nearby data together. |
| Example: Reading the same page multiple times. | Example: Accessing consecutive lines in a book. |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Temporal Locality] --> B{Example};
    B --> C[Accessing the same data repeatedly over short intervals];
    A --> D[Spacial Locality];
    D --> E{Example};
    E --> F[Accessing nearby data together];
```

**Example input / output:**

```text
**Temporal Locality Example:**
Input:
- Access sequence: `A -> B -> C -> A -> B -> C`

Output:
- Cache hits on accesses to `B` and `C`, due to recent access of `A`.

**Spatial Locality Example:**
Input:
- Memory addresses accessed: `[1, 2, 3]`

Output:
- Cache hit on accessing address `2` after `1`, since data at these consecutive addresses is likely related.
```

**Data:**

```json
{
  "temporalLocality": {
    "example": [
      { "timeInterval": "10ms", "dataAccessed": ["A", "B", "C"] }
    ]
  },
  "spatialLocality": {
    "example": [
      { "memoryAddresses": [1, 2, 3], "dataAccessed": ["X", "Y", "Z"] }
    ]
  }
}
```

**Flow:**

1. Repeatedly accessed soon
2. Not recently accessed
3. Nearby data accessed soon
4. Data not nearby
5. Start
6. Temporal Locality
7. Accessed Data
8. Cache Hit
9. Cache Miss
10. Start
11. Spatial Locality
12. Neighboring Addresses
13. Cache Hit
14. Cache Miss

