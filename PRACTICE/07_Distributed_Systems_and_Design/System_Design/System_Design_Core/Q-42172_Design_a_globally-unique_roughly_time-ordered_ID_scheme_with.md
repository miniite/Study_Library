---
type: "practice"
difficulty: "Deep"
---

# Design a globally-unique, roughly time-ordered ID scheme without a central counter. Compare Snowflake IDs, ULIDs, and UUIDv7 on index locality, clock-skew risk, and coordination cost.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clickstream Counting Pipeline](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a globally-unique, roughly time-ordered ID scheme without a central counter, we can compare Snowflake IDs, ULIDs, and UUIDv7 on key metrics such as index locality, clock skew risk, and coordination cost. Snowflake IDs are widely used for their performance and simplicity, while ULIDs aim to improve on this by incorporating human-readable timestamps and better locality. UUIDv7 offers flexibility but at a higher cost in terms of complexity and performance.

**Code:**

```python
# Example code snippets for generating these ID schemes
def generate_snowflake_id():
    # Snowflake ID generation logic here
    pass

def generate_ulid():
    # ULID generation logic here
    pass

def generate_uuidv7():
    # UUIDv7 generation logic here
    pass
```

**Data:**

```json
{
  "snowflake_id": "12345678901234567",
  "ulid": "01GSMQ4T9RSBZP5WJN86V0X4M6",
  "uuidv7": "d7c5e2b-6f2b-7ab8-d7c5-e2b6f2b7a8d9"
}
```

**Flow:**

1. No
2. Yes
3. Generate Snowflake ID
4. Has Global Clock?
5. Generate ULID
6. Generate UUIDv7
7. Check Index Locality
8. Evaluate Coordination Cost

