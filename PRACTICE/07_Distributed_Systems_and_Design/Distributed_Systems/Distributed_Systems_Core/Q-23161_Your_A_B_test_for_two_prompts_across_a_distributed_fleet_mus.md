---
type: "practice"
difficulty: "Intermediate"
---

# Your A/B test for two prompts across a distributed fleet must keep a given user on the same variant even as replicas scale. Explain hash-bucketing for sticky assignment and why non-sticky assignment silently invalidates your experiment stats.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Mixed Version Fleets](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of keeping a given user consistent across distributed A/B tests, we need to ensure that the same user is always assigned to the same variant, even as the system scales. This requirement is known as sticky assignment. Hash-bucketing is a technique used for achieving this consistency by mapping each user ID (or any unique identifier) to a specific bucket or variant group.

#### Triage
Understanding the Question

The question requires understanding why hash-bucketing ensures consistent user assignment and how non-sticky assignment can invalidate experiment statistics. We need to explain the core concepts of sticky assignment, the role of hash functions in distributing users across buckets, and the impact of non-sticky behavior on experiment validity.

#### First-Principles Solution
Step 1: Define Sticky Assignment

Sticky assignment ensures that a user is consistently assigned to the same variant group throughout their session or for a certain period. This is crucial because it allows us to accurately track user responses and gather meaningful statistical insights.

Step 2: Hash-Bucketing Explanation

Hash-bucketing involves using a hash function to map each user ID (or other unique identifier) to an integer bucket between 1 and the total number of variants. For example, if we have two variants (A and B), we can use modulo operation as follows:

PYTHON
variant = hash(user_id) % 2 + 1

Here, hash(user_id) generates a unique hash for each user ID, which is then reduced to an integer between 1 and the number of variants. The % operator ensures that the result falls within this range.

Step 3: Benefits of Hash-Bucketing

#### Cons
istency: Ensures that the same user always gets the same variant.
Scalability: Can be easily scaled by adding more buckets or variants without disrupting user assignments.
Efficiency: Uses simple modulo operations to determine bucket assignment, making it computationally efficient.
Step 4: Impact of Non-Sticky Assignment

Non-sticky assignment means that each request for a specific user can result in a different variant being assigned. This introduces variability and invalidates the experiment’s statistics because:

Loss of Consistency: Different requests from the same user might yield different results, making it hard to track their behavior over time.
Inaccurate Data Collection: The data collected will not reflect the true user experience since users are randomly reassigned to different variants on each request.

#### Closing Summary
and Common Mistakes

Hash-bucketing is a robust method for achieving sticky assignment in distributed systems. By using hash functions, we can ensure that each user remains consistently assigned to the same variant group. Non-sticky behavior, however, can severely impact the validity of A/B test results due to inconsistent user assignments.

#### Common Mistakes

Forgetting Hash Function Consistency: Using a non-deterministic or different hash function for each request.
Not Implementing Modulo Correctly: Failing to use modulo operations properly when reducing hash values to fit the number of variants.
Ignoring Scalability Issues: Assuming that sticky assignment will work without considering how adding more buckets or variants might affect performance.

**Comparison:**

| User ID | Variant Assignment |
|---------|--------------------|
| 100     | A                  |
| 200     | B                  |
| 300     | A                  |
| 400     | B                  |

**Code:**

```python
# Example Python code for hash-bucketing with two variants
def assign_variant(user_id):
    # Assume we use a simple hash function (this is just an example)
    variant = hash(user_id) % 2 + 1
    return variant
```

**Example input / output:**

```text
| User ID | Variant Assignment |
|---------|--------------------|
| 100     | A                  |
| 200     | B                  |
| 300     | A                  |
| 400     | B                  |

Here, the hash function is used to map each user ID to a variant. For simplicity, we assume that `hash(user_id) % 2 + 1` results in either 1 or 2.
```

**Data:**

```json
{
  "user_ids": [100, 200, 300, 400],
  "variants": []
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Sticky Assignment?
5. (Hash ID % Variants) + 1
6. Assign Variant to User
7. (Random or Non-Sticky)Assign Variant to User
8. Collect Data and AnalyzeResults
9. Data Collection May Be Invalid

