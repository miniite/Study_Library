---
type: "practice"
difficulty: "Expert"
---

# What's your strategy for blocking / candidate generation so you don't do O(n^2) comparisons at hundreds of millions of records — and how do you validate you're not losing true matches in the blocking step?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Records and Survivorship](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: When implementing Master Data Management (MDM), a key challenge is to efficiently block and generate candidate sets without performing (O(n^2)) comparisons on hundreds of millions of records. My strategy involves using a combination of hash-based blocking, probabilistic matching techniques, and validation checks to ensure accuracy while maintaining performance.

#### Triaging the Question
The question asks for an efficient strategy to block and generate candidates in MDM without performing exhaustive pairwise comparisons, which would be computationally expensive given large datasets. The goal is also to validate that no true matches are being missed during this process. This involves understanding the blocking criteria, candidate generation rules, and validation mechanisms.

Step-by-Step Solution from First Principles:

Define Blocking Criteria:

Use hash functions on key attributes (e.g., SSN, email) to create unique identifiers.

#### Example
PYTHON
import hashlib

def hash_key(key):
    return hashlib.md5(key.encode()).hexdigest()

Generate Candidate Sets:

Group records by these hashes and compare only within groups.
This reduces the number of comparisons from (O(n^2)) to (O(m \cdot k^2)), where (m) is the number of groups and (k) is the average size of each group.

Probabilistic Matching:

Implement techniques like fuzzy matching or using a probabilistic database (e.g., Probabilistic Databases in PostgreSQL).

#### Example
SQL
SELECT * FROM table1 t1, table2 t2 WHERE fuzzy_name_match(t1.name, t2.name) > 0.8;

Validation Mechanisms:

Implement a two-pass validation approach: first, perform candidate generation and then validate matches.

#### Example
PYTHON
def validate_matches(candidate_pairs):
    true_positives = set()
    for pair in candidate_pairs:
        if verify_pair(pair[0], pair[1]):
            true_positives.add(pair)
    return true_positives

Monitoring and Optimization:

Continuously monitor the blocking criteria effectiveness.
Adjust hash functions, thresholds, or even re-calculate hashes as needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes

Overly strict blocking rules leading to false negatives.
Ignoring edge cases and not having robust validation mechanisms.
Not continuously monitoring the effectiveness of blocking criteria.

#### Follow-Ups

How do you handle real-time data?
What about dealing with unstructured or semi-structured data?

**Comparison:**

```
Record A	Record B
John Doe	Jane Smith
True Positive
(John Doe, Jane Smith)
```

**Code:**

```python
import hashlib

def hash_key(key):
    return hashlib.md5(key.encode()).hexdigest()

# Example of candidate generation and validation
class MDM:
    def __init__(self, records):
        self.records = records
        self.blocked_records = {}

    def block_records(self):
        for record in self.records:
            key = hash_key(record.key_attribute)
            if key not in self.blocked_records:
                self.blocked_records[key] = []
            self.blocked_records[key].append(record)

    def generate_candidates(self):
        candidate_pairs = set()
        for group in self.blocked_records.values():
            for i, record1 in enumerate(group):
                for j, record2 in enumerate(group):
                    if i < j:
                        candidate_pairs.add((record1, record2))
        return candidate_pairs

def validate_matches(candidate_pairs):
    true_positives = set()
    for pair in candidate_pairs:
        if verify_pair(pair[0], pair[1]):
            true_positives.add(pair)
    return true_positives
```

**Example input / output:**

```text
records = [
    {"id": 1, "name": "John Doe", "email": "johndoe@example.com"},
    {"id": 2, "name": "Jane Smith", "email": "janesmith@example.org"},
    # ... hundreds of millions more records ...
]

mdm = MDM(records)
mdm.block_records()
candidate_pairs = mdm.generate_candidates()

true_positives = validate_matches(candidate_pairs)
print(true_positives)
```

**Data:**

```json
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "johndoe@example.com",
    "ssn": "123-45-6789"
  },
  {
    "id": 2,
    "name": "Jane Smith",
    "email": "janesmith@example.org",
    "ssn": "987-65-4321"
  }
]
```

**Flow:**

1. Yes
2. Yes
3. No
4. Start
5. Is record count > 100M?
6. Hash and Block records
7. Generate candidate pairswithin blocks
8. Validate matches
9. True positive identified
10. Re-evaluate blocking rules

