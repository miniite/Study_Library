---
type: "practice"
difficulty: "Deep"
---

# You need an advisory lock keyed on a 128-bit business id but advisory locks take a bigint; how would you derive a stable 64-bit key from a UUID and what collision risk do you accept?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ UUID vs Serial Keys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/UUID_vs_Serial_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to map a 128-bit UUID (universally unique identifier) into a stable 64-bit integer that can be used as an advisory lock key in PostgreSQL. The goal is to ensure minimal collisions while adhering to the constraints of an advisory_lock mechanism.

#### Triage

The question essentially asks how to convert a UUID, which is represented as a 128-bit integer, into a smaller 64-bit value that can be used for advisory locks. PostgreSQL's advisory_lock API requires a bigint (which fits within 64 bits), so we need to derive this key from the larger UUID.

Key Idea

UUIDs are designed to be globally unique, but converting them into a smaller integer space will introduce a risk of collisions. We'll use a hash function or a deterministic algorithm to reduce the UUID size while minimizing the chance of collisions.

Step-by-Step Solution

Understand UUID Structure: A UUID is typically represented as 32 hexadecimal characters (16 bytes) when stored in text form, but it can be treated as an integer for this purpose.

Choose a Hash Function or Algorithm:

We need to ensure the algorithm is deterministic and produces consistent results.
SHA-1 or SHA-256 are strong cryptographic hash functions that could work well here.

Extract and Convert UUID:

For simplicity, let's use the first 8 bytes of the UUID (64 bits) as our starting point. This can be done by extracting the lower 64 bits from the full 128-bit UUID.
Convert these 8 bytes into a 64-bit integer.

Apply Hash Function:

Use SHA-256 to hash the first 8 bytes of the UUID and extract the least significant 64 bits of the resulting hash as our key.
PYTHON
import uuid
from hashlib import sha256

def derive_lock_key(uuid_str: str) -> int:
    # Convert UUID string to bytes (16 bytes)
    uuid_bytes = uuid.UUID(uuid_str).bytes

    # Extract first 8 bytes (64 bits)
    lower_64_bits = int.from_bytes(uuid_bytes[:8], byteorder='big')

    # Hash the lower 64 bits and extract the least significant 64 bits
    hash_value = sha256(lower_64_bits.to_bytes(8, 'big')).hexdigest()
    lock_key = int(hash_value, 16) & ((1 << 64) - 1)

    return lock_key

# Example UUID and derived key

#### example
_uuid = "123e4567-e89b-12d3-a456-426614174000"
derived_key = derive_lock_key(example_uuid)
print(f"Derived Lock Key: {derived_key}")
Collision Risk

While the approach minimizes collisions, they are not entirely eliminated. The use of SHA-256 to hash the first 8 bytes and then extract the least significant 64 bits reduces the chance of collisions but does not guarantee uniqueness across all UUIDs.

#### Closing Summary

The solution involves using a cryptographic hash function like SHA-256 to reduce a 128-bit UUID into a stable 64-bit integer suitable for advisory locks. This approach balances between reducing collision risk and leveraging the deterministic nature of hashes.

#### Common Mistakes/Follow-ups
Incorrect Hashing: Using an inappropriate hashing algorithm or not properly handling byte conversions.
Ignoring Collision Risk: Failing to acknowledge that while collisions are minimized, they cannot be completely avoided.
Code Example
PYTHON
import uuid
from hashlib import sha256

def derive_lock_key(uuid_str: str) -> int:
    # Convert UUID string to bytes (16 bytes)
    uuid_bytes = uuid.UUID(uuid_str).bytes

    # Extract first 8 bytes (64 bits)
    lower_64_bits = int.from_bytes(uuid_bytes[:8], byteorder='big')

    # Hash the lower 64 bits and extract the least significant 64 bits
    hash_value = sha256(lower_64_bits.to_bytes(8, 'big')).hexdigest()
    lock_key = int(hash_value, 16) & ((1 << 64) - 1)

    return lock_key

# Example UUID and derived key

#### example
_uuid = "123e4567-e89b-12d3-a456-426614174000"
derived_key = derive_lock_key(example_uuid)
print(f"Derived Lock Key: {derived_key}")

**Code:**

```python
import uuid
from hashlib import sha256

def derive_lock_key(uuid_str: str) -> int:
    # Convert UUID string to bytes (16 bytes)
    uuid_bytes = uuid.UUID(uuid_str).bytes

    # Extract first 8 bytes (64 bits)
    lower_64_bits = int.from_bytes(uuid_bytes[:8], byteorder='big')

    # Hash the lower 64 bits and extract the least significant 64 bits
    hash_value = sha256(lower_64_bits.to_bytes(8, 'big')).hexdigest()
    lock_key = int(hash_value, 16) & ((1 << 64) - 1)

    return lock_key

# Example UUID and derived key
example_uuid = "123e4567-e89b-12d3-a456-426614174000"
derived_key = derive_lock_key(example_uuid)
print(f"Derived Lock Key: {derived_key}")
```

**Example input / output:**

```
Example input and output

example_uuid = "123e4567-e89b-12d3-a456-426614174000" derived_key = derive_lock_key(example_uuid) print(f"Derived Lock Key: {derived_key}")

CODE
Copy

Output:

Derived Lock Key: 30645518579001065
```

