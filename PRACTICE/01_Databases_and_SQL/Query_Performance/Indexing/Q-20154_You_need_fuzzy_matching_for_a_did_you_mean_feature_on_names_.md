---
type: "practice"
difficulty: "Deep"
---

# You need fuzzy matching for a 'did you mean' feature on names. Compare a trigram index, precomputed phonetic codes, and delegating to a search service on cost and latency.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Indexing Read State at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Indexing_Read_State_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of implementing a "did you mean" feature using fuzzy matching, we need to consider three main approaches: trigram indexing, precomputed phonetic codes, and delegating to a search service. Each method has its own cost implications and latency characteristics, which we will analyze from first principles.

First, let's understand what each approach entails:

Trigram Indexing: This involves breaking down words into three-letter segments (trigrams) and indexing them for quick comparison.
Precomputed Phonetic Codes: Using algorithms like Soundex or Metaphone to generate phonetic representations of words and compare them.
Delegating to a Search Service: Leverage services that specialize in fuzzy matching, such as Elasticsearch.

Now, let's break down the cost and latency considerations for each approach:

Trigram Indexing
Cost Implications:
Storage Overhead: Each word is broken into trigrams, leading to increased storage requirements.
Index Maintenance: Regular updates are necessary when new data is added or existing data changes.
Latency Considerations:
Query Processing: Fast for exact matches but slower for fuzzy matching due to the need to compare multiple trigram segments.
Scalability: Can be efficiently managed with database optimizations, but performance degrades with large datasets.
Precomputed Phonetic Codes
Cost Implications:
Storage Overhead: Similar to trigrams, additional storage is required for phonetic codes.
Algorithm Complexity: The process of generating and storing these codes can introduce some overhead during initial setup.
Latency Considerations:
Query Processing: Faster than trigram indexing since comparisons are based on precomputed codes.
Scalability: More scalable due to the reduction in comparison complexity, but still dependent on the quality and coverage of the phonetic algorithm used.
Delegating to a Search Service
Cost Implications:
API Usage Costs: Depend on the usage policies of the search service provider.
Infrastructure Overhead: Hosting additional infrastructure for integration and management.
Latency Considerations:
Query Processing: Highly optimized by specialized services, leading to very fast response times.
Scalability: Scalable as the service handles load balancing and performance optimization.
Summary

From a cost perspective, trigram indexing might be more straightforward but could become expensive with large datasets. Precomputed phonetic codes offer better scalability at slightly higher initial costs. Delegating to a search service provides the best balance of cost and latency for most use cases but comes with API usage fees.

For a "did you mean" feature in a database setting, precomputed phonetic codes are often a good middle ground. They provide acceptable performance while minimizing the storage and maintenance overhead compared to trigram indexing. If budget constraints or high traffic levels are critical concerns, delegating to a search service can offer superior performance at scale.

#### Common Mistakes/Follow-Ups
Not Considering Index Maintenance: Ignoring the ongoing costs of maintaining an index.
Overlooking Scalability Issues: Failing to account for how each approach scales with increasing data volumes.
Ignoring API Costs: Not factoring in the long-term expenses associated with using a third-party service.

**Comparison:**

```
Approach	Cost Implications	Latency Considerations
Trigram	Higher storage and maintenance costs	Slower for fuzzy matching
Phonetic	Lower initial cost, good scalability	Fast query processing
Search	API usage fees	Very fast response times
```

**Code:**

```

-- Example SQL for Trigram Indexing CREATE INDEX idx_name_trigrams ON names USING GIN (to_tsvector('english', name));

CODE
Copy

```python
# Example Python code for Phonetic Code Generation
import phonet

def generate_phonetic_code(word):
    return phonet.encode(word)

# Example usage
phonetic_codes = [generate_phonetic_code(name) for name in names]
```

**Example input / output:**

```text
Input:
- Query: "Smit"

Output:
- Possible Matches: ["Smith", "Smyth"]
- Reasoning: Using phonetic codes, the system identified words with similar sound patterns.
```

**Data:**

```json
{
  "names": [
    {"id": 1, "name": "Smith"},
    {"id": 2, "name": "Smyth"},
    {"id": 3, "name": "Smitt"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Query
6. Is Trigram Index Available?
7. Generate Matches
8. Use Phonetic Codes
9. Are Matches Sufficient?
10. Return Matches
11. Delegate to Search Service

