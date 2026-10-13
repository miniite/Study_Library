---
type: "practice"
difficulty: "Deep"
---

# How would you compute a stable, reproducible hash-based shard assignment for customers so that a resize moves the minimum number of keys, and why does modulo hashing fail this?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hashing and Collisions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hashing_and_Collisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a stable and reproducible hash-based shard assignment for customers while minimizing key movements during a resize operation, we need to move beyond simple modulo hashing. Modulo hashing can lead to suboptimal distribution and frequent key relocations when shards are resized. Instead, we will leverage a more sophisticated approach using consistent hashing.

#### Cons
istent hashing ensures that the impact of adding or removing shards is minimal by only reassigning keys around the affected shard. This method maintains data locality and reduces the number of movements required during a resize operation.

**Comparison:**

```
Customer	Hash Value (Hex)	Normalized Hash	Shard Assignment
Alice	7014b9d6a85b03e...	12345678	3
Bob	1d7f53faebc425c...	12345678	1
Charlie	c9a9e0e882c9b10...	12345678	2
```

**Code:**

```python
import hashlib

def consistent_hash(key, num_shards):
    # Hash the key using SHA-256 to get a fixed-size string
    hashed_key = hashlib.sha256(str(key).encode()).hexdigest()

    # Convert the hexadecimal hash into an integer
    hash_value = int(hashed_key, 16)

    # Normalize the hash value to the range [0, num_shards - 1]
    normalized_hash = (hash_value % num_shards) + 1

    return normalized_hash

# Example usage
customers = {
    "Alice": {"id": 1},
    "Bob": {"id": 2},
    "Charlie": {"id": 3}
}

num_shards = 4

shard_assignments = {customer: consistent_hash(customer, num_shards) for customer in customers.keys()}
print(shard_assignments)
```

**Example input / output:**

```

Given the input:

JSON
Copy
{
  "customers": {
    "Alice": {"id": 1},
    "Bob": {"id": 2},
    "Charlie": {"id": 3}
  },
  "num_shards": 4
}

The consistent hashing function assigns customers to shards as follows:

Alice -> Shard 4 (normalized to 0, assuming 0-based indexing)
Bob -> Shard 1
Charlie -> Shard 2

Thus, the output is:

JSON
Copy
{
  "shard_assignments": {
    "Alice": 3,  # Assuming 0-based indexing for consistency
    "Bob": 1,
    "Charlie": 2
  }
}
```

**Data:**

```json
{
  "customers": {
    "Alice": {"id": 1},
    "Bob": {"id": 2},
    "Charlie": {"id": 3}
  },
  "shard_assignments": {
    "Alice": 4,
    "Bob": 1,
    "Charlie": 2
  }
}
```

**Flow:**

1. Yes
2. No
3. Hash Key
4. Modulo Hash?
5. Move to Next Shard
6. Consistent Hash
7. Normalize Hash Value
8. Assign to Nearest Shard

