---
type: "practice"
difficulty: "Deep"
---

# What is fuzzy matching and when would you use it to detect near duplicate customer records?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Uniqueness Checks](<../../../../02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Fuzzy matching is a technique used in data quality and validation to identify near-duplicate records based on similarity rather than exact equality. It's particularly useful when dealing with noisy or inconsistently formatted data, such as customer names, addresses, or phone numbers.

To effectively use fuzzy matching, let's break down the question into its core components:

Understanding Fuzzy Matching: Fuzzy matching involves comparing strings to determine how similar they are, even if they don't match exactly.
Identifying Near-Duplicate Records: This is crucial in data quality to ensure that records with slight variations (e.g., misspellings, abbreviations) are not incorrectly treated as distinct entities.
Step-by-Step Solution

Define the Problem Context:

Customer Data: In this scenario, we have customer records containing names, addresses, and phone numbers.
Data Variability: These fields can contain variations like typos, abbreviations, or inconsistent formatting (e.g., John Doe vs. J. D.).

Choose a Fuzzy Matching Algorithm:

Common algorithms include Levenshtein Distance, Jaccard Similarity, and Soundex.
For simplicity, we'll use the Levenshtein Distance to measure the number of single-character edits (insertions, deletions, or substitutions) required to change one word into another.

Set a Threshold for Near-Duplicates:

Determine a threshold value that defines when two records are considered near-duplicate.
For instance, if the Levenshtein Distance is less than 2, we consider the records as potential duplicates.

Implement the Fuzzy Matching Logic:

Compare all possible pairs of customer records for each field (name, address, phone number).
If any pair meets the threshold criteria, flag them as near-duplicates.

#### Example
For John Doe and J. D. in the name field, calculate their Levenshtein Distance.

Validate and Cleanse Data:

Apply fuzzy matching to a sample of data to validate its effectiveness.

#### Cons
ider additional cleaning steps like normalization (e.g., removing punctuation) before applying fuzzy matching.

#### Example
Walkthrough

Let's walk through an example with the following customer records:

Record 1: John Doe, 123 Main St, 555-555-5555
Record 2: J. D., 123 Main Street, 555-555-5555
Step-by-Step Calculation:

Normalize the Data:

Remove punctuation and standardize spaces.
John Doe → john doe
123 Main St → 123 main street

Calculate Levenshtein Distance for Names:

john doe vs. j d:
Insertions: 0
Deletions: 0
Substitutions: 2 (replace 'o' with nothing, replace 'e' with nothing)
Total: 2

Evaluate Other Fields:

Address: 123 main street vs. 123 main st
Insertions: 0
Deletions: 0
Substitutions: 0 (no difference)
Total: 0 < threshold

Final Decision:

Since the name field has a Levenshtein Distance of 2, and the address is exact, we flag these records as near-duplicates.

#### Closing Summary
and Common Mistakes/Follow-Ups

Summary: Fuzzy matching is essential for detecting near-duplicate customer records in noisy data environments. By implementing algorithms like Levenshtein Distance with a set threshold, you can identify and clean up data more effectively.

#### Common Mistakes

Over-relying on fuzzy matching without normalization or additional cleaning steps.
Setting overly strict thresholds that fail to capture legitimate near-duplicates.
Ignoring other fields beyond the primary key (e.g., phone number).

#### Follow-Ups
Consider integrating machine learning models for better accuracy, especially with large datasets. Explore other string similarity algorithms and their trade-offs.

**Comparison:**

| Record 1 | Record 2 | Name Distance |
|----------|----------|---------------|
| John Doe | J. D.    | 2             |

**Code:**

```python
def levenshtein_distance(s1, s2):
    if len(s1) < len(s2):
        return levenshtein_distance(s2, s1)

    # len(s1) >= len(s2)
    if len(s2) == 0:
        return len(s1)

    previous_row = range(len(s2) + 1)
    for i, c1 in enumerate(s1):
        current_row = [i + 1]
        for j, c2 in enumerate(s2):
            insertions = previous_row[j + 1] + 1
            deletions = current_row[j] + 1
            substitutions = previous_row[j] + (c1 != c2)
            current_row.append(min(insertions, deletions, substitutions))
        previous_row = current_row

    return previous_row[-1]

# Example usage
record_1_name = "John Doe"
record_2_name = "J. D."
distance = levenshtein_distance(record_1_name.lower().replace(" ", ""), record_2_name.lower().replace(" ", ""))
print(f"Levenshtein Distance: {distance}")
```

**Example input / output:**

```text
# Example Input and Output
records = [
    {"name": "John Doe", "address": "123 Main St", "phone": "555-555-5555"},
    {"name": "J. D.", "address": "123 Main Street", "phone": "555-555-5555"}
]

threshold = 2

# Flagging near-duplicates
for i, record_1 in enumerate(records):
    for j, record_2 in enumerate(records[i+1:], start=i+1):
        if levenshtein_distance(record_1['name'].lower().replace(" ", ""), record_2['name'].lower().replace(" ", )) < threshold:
            print(f"Near-duplicate found: {record_1} and {record_2}")
```

**Data:**

```json
{
  "records": [
    {"name": "John Doe", "address": "123 Main St", "phone": "555-555-5555"},
    {"name": "J. D.", "address": "123 Main Street", "phone": "555-555-5555"}
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Normalize Data
7. Is Field Similar?
8. Calculate Distance
9. Threshold Met?
10. Mark as Near-Duplicate
11. Continue to Next Pair
12. Skip This Pair
13. End

