---
type: "practice"
difficulty: "Deep"
---

# You must support fuzzy matching for a 'did you mean' feature over usernames. Compare a trigram index, precomputed phonetic codes, and an external search service on latency and index size.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Indexing Read State at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Indexing_Read_State_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of implementing a "did you mean" feature using fuzzy matching for usernames, we need to compare three potential approaches: trigram indexing, precomputed phonetic codes, and an external search service. Let's break down each method in terms of latency and index size.

#### Triage

The core question here is to evaluate the trade-offs between different methods of implementing a fuzzy match feature over usernames. We'll consider both the performance (latency) and storage requirements (index size).

Key Assumptions:
Usernames: These are typically strings of varying lengths, often used in login systems.
Fuzzy Matching Algorithm: A trigram-based algorithm will be assumed for simplicity, though other algorithms like Levenshtein distance or Soundex could also be relevant.
Latency Requirements: The system needs to provide fast responses, ideally within 100ms per query.
Index Size: The size of the indexes should not excessively impact storage costs.

#### First-Principles Solution
Trigram Indexing

A trigram index works by breaking each username into three-character segments (trigrams) and storing these in an index. For example, "username" would generate trigrams like win, ini, etc.

Latency:

#### Pros
Fast to query as it's a simple substring match.

#### Cons
May return many false positives due to the nature of substring matching (e.g., "usern" could be similar to "username", but also "usurper").

Index Size:

Each username generates multiple trigrams, leading to a large index size.
For usernames like "superuser" and "supervisor," there would be significant overlap in the trigram sets.
Precomputed Phonetic Codes

Phonetic codes convert words into phonetic representations (like Soundex or Metaphone). This can help identify similarly sounding words, which are often what users mean when they make a typo.

Latency:

#### Pros
Very fast to compare phonetic codes as it's typically just a string comparison.

#### Cons
Requires precomputation of the phonetic codes for all usernames, which can take significant time and resources.

Index Size:

The index size is smaller compared to trigram indexing since each username maps to a single code (or at most a few).
External Search Service

Using an external search service like Elasticsearch or Solr that supports fuzzy matching out-of-the-box could be another option.

Latency:

#### Pros
Provides advanced features and fast responses, often optimized for large datasets.

#### Cons
Involves network latency and may have higher costs associated with using an external service.

Index Size:

The index size depends on the configuration of the search service but is generally managed by the service provider, reducing the need for local storage management.

#### Closing Summary

In summary, trigram indexing offers fast substring matching at the cost of potentially many false positives and large index sizes. Precomputed phonetic codes provide a more accurate match with lower false positives but require significant precomputation time. An external search service can offer advanced features and faster responses but introduces network latency and costs.

#### Common Mistakes/Follow-Ups
Ignoring False Positives: Focusing solely on fast queries without considering the accuracy of results.
Overlooking External Costs: Not accounting for the potential overhead in using an external service.
Assuming Constant Latency: Overestimating the consistency of latency across different methods.
Depth and Language Match the Level

This deep understanding requires a nuanced evaluation of each method, balancing performance and storage requirements. The solution considers various factors like false positives, precomputation costs, network latency, and external service usage.

**Comparison:**

```
Method	Latency (Best/Worst)	Index Size
Trigram Indexing	Fast/Variable	Large
Precomputed Codes	Fast	Small
External Search	Variable	Managed
```

**Code:**

```python
# Example Trigram Indexing Code (simplified)
def generate_trigrams(username):
    return [username[i:i+3] for i in range(len(username) - 2)]

trigrams = generate_trigrams("superuser")
print(trigrams)  # Output: ['sup', 'upe', 'per', 'ser', 'sur', 'rue', 'uer', 'eru']
```

**Example input / output:**

```text
Input:
- Username: "superuser"

Output (Trigram Index):
- ['sup', 'upe', 'per', 'ser', 'sur', 'rue', 'uer', 'eru']

Phonetic Code Example:
- Soundex("superuser"): "S182"
```

**Data:**

```json
{
  "username": "superuser",
  "trigram_index": ["sup", "upe", "per", "ser", "sur", "rue", "uer", "eru"]
}
```

**Flow:**

1. User Query
2. Trigram Matching
3. False Positives Possible
4. Phonetic Code Lookup
5. Exact Matches Found
6. External Search Query
7. Advanced Fuzzy Matching

