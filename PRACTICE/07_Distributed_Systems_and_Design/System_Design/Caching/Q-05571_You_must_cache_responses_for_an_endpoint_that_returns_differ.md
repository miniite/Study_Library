---
type: "practice"
difficulty: "Deep"
---

# You must cache responses for an endpoint that returns different data based on 12 query parameters, most of which are irrelevant to the result. How do you avoid a combinatorial key explosion?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Caching Expensive Queries](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of caching responses from an endpoint with 12 query parameters where only a few are relevant, we need to design a caching strategy that minimizes the number of unique cache keys while still ensuring we get the correct results. The core idea is to hash or filter out irrelevant parameters and group similar data into buckets.

#### Triage

The question is asking how to manage caching for an endpoint with 12 query parameters, most of which are irrelevant to the final result. The interviewer is probing our understanding of cache key optimization and how to handle a large number of potential unique keys efficiently without compromising correctness.

#### Assumptions
The 12 query parameters can be categorized into relevant (affect the result) and irrelevant ones.
We need to ensure that only one version of each combination of relevant parameter values is cached.

#### First-Principles Solution
Identify Relevant Parameters: Determine which parameters affect the final result. Let's assume there are 3 such parameters out of 12, named paramA, paramB, and paramC.
Combine Relevant Parameters: Use a hash function to combine these relevant parameter values into a unique key.
Bucket Irrelevant Parameters: Group similar irrelevant parameter values together to minimize the number of keys.
Detailed Steps

Hash Combination:

Extract the 3 relevant parameters from the query string: paramA, paramB, and paramC.
Combine them using a hash function (e.g., SHA-256) to create a unique key.

Bucket Irrelevant Parameters:

Group similar irrelevant parameter values into buckets. For example, if there are 8 irrelevant parameters (param4 to param11), we can use their values in conjunction with the relevant parameters to decide on which bucket to place them.

Caching Strategy:

Cache the combined key with the response for each unique combination of relevant parameter values.
When a request comes in, check if it matches any of the existing keys; if so, return the cached result.
If no exact match exists, use the hashed key and bucketed irrelevant parameters to decide on which cache entry (if any) to use.

#### Example

Let’s assume we have 3 relevant parameters: paramA, paramB, paramC. We hash these values together:

PYTHON
import hashlib

def generate_cache_key(paramA, paramB, paramC):
    # Combine the relevant parameters and hash them
    combined = f"{paramA}-{paramB}-{paramC}"
    return hashlib.sha256(combined.encode()).hexdigest()

For irrelevant parameters, we can group them into buckets:

PYTHON
def bucket_irrelevant_params(params):
    # Example: param4 to param11 are grouped by their values
    hash_bucket = params['param4'] + params['param5']
    return hash_bucket

# Example usage
params = {
    'paramA': 'valueA',
    'paramB': 'valueB',
    'paramC': 'valueC',
    # Irrelevant parameters
    'param4': 'value4-1',
    'param5': 'value5-2',
    # ... other irrelevant params
}

cache_key = generate_cache_key(params['paramA'], params['paramB'], params['paramC'])
bucketed_params = bucket_irrelevant_params(params)

# Use cache_key and bucketed_params to look up in the cache

#### Closing Summary

By hashing relevant parameters and grouping irrelevant ones into buckets, we can significantly reduce the number of unique cache keys. This approach ensures that only one version of each combination of relevant parameter values is cached, while still handling similar irrelevant configurations efficiently.

#### Common Mistakes/Follow-Ups
Not Identifying Relevant Parameters: Failing to distinguish between parameters that affect the result and those that do not can lead to unnecessary complexity.
Over-Hashing: Hashing too many or irrelevant parameters can increase the key space unnecessarily.
Ignoring Bucketing Logic: Without proper bucketing logic, similar irrelevant configurations might cause cache misses.

#### Follow-up
questions could include:

How would you handle edge cases where parameter values are null or default?
What if the number of relevant parameters changes over time?
How do you ensure that the cache is updated when any of these parameters change?

**Code:**

```python
import hashlib

def generate_cache_key(paramA, paramB, paramC):
    combined = f"{paramA}-{paramB}-{paramC}"
    return hashlib.sha256(combined.encode()).hexdigest()

def bucket_irrelevant_params(params):
    # Example: param4 to param11 are grouped by their values
    hash_bucket = params['param4'] + params['param5']
    return hash_bucket

# Example usage
params = {
    'paramA': 'valueA',
    'paramB': 'valueB',
    'paramC': 'valueC',
    # Irrelevant parameters
    'param4': 'value4-1',
    'param5': 'value5-2',
    # ... other irrelevant params
}

cache_key = generate_cache_key(params['paramA'], params['paramB'], params['paramC'])
bucketed_params = bucket_irrelevant_params(params)
print(f"Cache Key: {cache_key}, Bucketed Params: {bucketed_params}")
```

**Flow:**

1. Yes
2. No
3. Request comes in
4. Is cache key present?
5. Use cached result and return
6. Generate cache key, updatebucketed params, store incache if not exists then return

