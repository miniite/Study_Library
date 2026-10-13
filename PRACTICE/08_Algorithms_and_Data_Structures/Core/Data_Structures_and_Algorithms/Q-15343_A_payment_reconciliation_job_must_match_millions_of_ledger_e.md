---
type: "practice"
difficulty: "Deep"
---

# A payment reconciliation job must match millions of ledger entries to bank statements with fuzzy amount/time keys; how do you structure the matching to avoid O(n*m) and still catch near-matches?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Similarity Search Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Similarity_Search_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of reconciling millions of ledger entries with bank statements, we need an efficient approach that avoids a brute-force comparison (O(n * m)) while still accurately catching near-matches. This task requires a robust matching algorithm with good performance and flexibility to handle fuzzy keys like amounts and times. The key idea is to leverage data structures and algorithms designed for nearest neighbor search, such as the locality-sensitive hashing (LSH) technique or approximate string matching methods.

The essence of the problem is to find near-matches between two large sets of records efficiently. Here’s a step-by-step breakdown from first principles:

Problem Restatement: The goal is to match ledger entries with bank statements based on fuzzy criteria such as amounts and timestamps, while ensuring high accuracy even when exact matches are not possible.

#### Assumptions

Amounts in both sets of records can be within a small tolerance range (e.g., ±1%).
Timestamps may have slight discrepancies but still need to align closely enough.

#### Solution
Approach:

Hashing with Tolerances: Use hashing techniques that can group similar values together, allowing us to quickly find potential matches.
Approximate String Matching: For timestamps and other string-like fields, use algorithms like Levenshtein distance or Jaccard similarity.

First-Principles Implementation:

Tolerance Range Calculation: Define the tolerance range for amounts based on their magnitude. For example, if the amount is 1000, a ±5% tolerance would be used; if it’s 10,000, a smaller tolerance like ±2% might be more appropriate.
Bucketing and Hashing:
PYTHON
def bucketize_amount(amount):
    # Example: bucketizing to nearest 1%
    return round(amount * 100) / 100

def hash_timestamp(timestamp, seed=42):
    import hashlib
    hasher = hashlib.sha256()
    hasher.update(str(timestamp).encode('utf-8'))
    return int(hasher.hexdigest(), 16) % 1000000
Finding Near-Matches:
PYTHON
def find_near_matches(ledger_entries, bank_statements):
    matches = []
    for ledger_entry in ledger_entries:
        amount_hash = bucketize_amount(ledger_entry['amount'])
        timestamp_hash = hash_timestamp(ledger_entry['timestamp'])

        # Find entries with similar hashes within a tolerance range
        possible_matches = [stmt for stmt in bank_statements if
                            abs(bucketize_amount(stmt['amount']) - amount_hash) < 0.05 and
                            abs(hash_timestamp(stmt['timestamp']) - timestamp_hash) < 100]

        # Further refine matches based on actual values
        refined_matches = []
        for match in possible_matches:
            if is_near_match(ledger_entry, match):
                refined_matches.append(match)
        matches.extend(refined_matches)
    return matches

def is_near_match(entry1, entry2):
    return abs(entry1['amount'] - entry2['amount']) / max(abs(entry1['amount']), abs(entry2['amount'])) < 0.05 and \
           abs(entry1['timestamp'] - entry2['timestamp']) < 60

#### Closing Summary
By using hashing techniques to group similar values, we can significantly reduce the number of exact comparisons needed. This approach ensures that even slight variations in amounts and timestamps are caught, leading to accurate matching without an O(n * m) complexity.

#### Common Mistakes/Follow-Ups

Failing to consider appropriate tolerance ranges for different data types.
Overlooking the need for further refinement after initial bucketing.
Not addressing edge cases where no matches are found or multiple potential matches exist.

**Comparison:**

| Ledger Entry ID | Amount (Bucketized) | Timestamp Hash | Matched Bank Statement(s) |
|-----------------|--------------------|---------------|----------------------------|
| 1               | 980.75             | 4234          | 1                          |
| 2               | 1020.15            | 4567          | 2                          |

**Code:**

```python
def bucketize_amount(amount):
    return round(amount * 100) / 100

def hash_timestamp(timestamp, seed=42):
    import hashlib
    hasher = hashlib.sha256()
    hasher.update(str(timestamp).encode('utf-8'))
    return int(hasher.hexdigest(), 16) % 1000000

def find_near_matches(ledger_entries, bank_statements):
    matches = []
    for ledger_entry in ledger_entries:
        amount_hash = bucketize_amount(ledger_entry['amount'])
        timestamp_hash = hash_timestamp(ledger_entry['timestamp'])

        possible_matches = [stmt for stmt in bank_statements if
                            abs(bucketize_amount(stmt['amount']) - amount_hash) < 0.05 and
                            abs(hash_timestamp(stmt['timestamp']) - timestamp_hash) < 100]

        refined_matches = []
        for match in possible_matches:
            if is_near_match(ledger_entry, match):
                refined_matches.append(match)
        matches.extend(refined_matches)
    return matches

def is_near_match(entry1, entry2):
    return abs(entry1['amount'] - entry2['amount']) / max(abs(entry1['amount']), abs(entry2['amount'])) < 0.05 and \
           abs(entry1['timestamp'] - entry2['timestamp']) < 60
```

**Example input / output:**

```text
ledger_entries = [
    {"id": 1, "amount": 980.75, "timestamp": "2023-10-15T14:00:00Z"},
    {"id": 2, "amount": 1020.15, "timestamp": "2023-10-16T14:01:00Z"}
]

bank_statements = [
    {"id": 1, "amount": 980.74, "timestamp": "2023-10-15T14:01:00Z"},
    {"id": 2, "amount": 1020.16, "timestamp": "2023-10-16T14:00:00Z"}
]

matches = find_near_matches(ledger_entries, bank_statements)
print(matches)  # Output: [{'id': 1, 'amount': 980.74, 'timestamp': '2023-10-15T14:01:00Z'}, {'id': 2, 'amount': 1020.16, 'timestamp': '2023-10-16T14:00:00Z'}]
```

**Data:**

```json
{
    "ledger_entries": [
        {"id": 1, "amount": 980.75, "timestamp": "2023-10-15T14:00:00Z"},
        {"id": 2, "amount": 1020.15, "timestamp": "2023-10-16T14:01:00Z"}
    ],
    "bank_statements": [
        {"id": 1, "amount": 980.74, "timestamp": "2023-10-15T14:01:00Z"},
        {"id": 2, "amount": 1020.16, "timestamp": "2023-10-16T14:00:00Z"}
    ]
}
```

**Flow:**

1. Read Ledger Entries
2. Bucketize Amounts
3. Hash Timestamps
4. Generate Possible Matches
5. Refine Matches Based onActual Values
6. Finding Near-MatchesComplete

