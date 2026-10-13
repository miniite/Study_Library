---
type: "practice"
difficulty: "Deep"
---

# Explain how HBase region hotspotting happens on a monotonically increasing rowkey and how you would design the rowkey to avoid it.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ YARN Troubleshooting](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/YARN_Troubleshooting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Region hotspotting in HBase occurs when data is inserted or accessed with a monotonically increasing row key, causing all writes or reads to be concentrated on a single region server. This imbalance can lead to performance bottlenecks and even failures due to overloaded servers. To avoid this, we need to design the rowkey effectively by incorporating a timestamp, user ID, or other factors that introduce variability.

**Comparison:**

```
Rowkey	User ID	Timestamp
u12345_20230901	u12345	20230901
u12346_20230901	u12346	20230901
u12347_20230902	u12347	20230902
```

**Code:**

```python
# Example of a simple rowkey with a timestamp and user ID
def create_rowkey(user_id, timestamp):
    return f"{user_id}_{timestamp}"

rowkeys = [create_rowkey("u12345", 20230901), create_rowkey("u12346", 20230901), create_rowkey("u12347", 20230902)]
```

**Example input / output:**

```text
Given the rowkeys generated above, we can see how they distribute across different region servers. For example:
- `u12345_20230901` maps to one server
- `u12346_20230901` maps to another (assuming user IDs and timestamps are well-distributed)

This would prevent hotspotting as the data is spread out across multiple regions.
```

**Data:**

```json
{
  "rowkeys": [
    {"user_id": "u12345", "timestamp": 20230901},
    {"user_id": "u12346", "timestamp": 20230901},
    {"user_id": "u12347", "timestamp": 20230902}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Rowkey Creation
4. Is User ID and TimestampUnique?
5. Generate Rowkey with User IDand Timestamp
6. Use Additional Factors toIncrease Variability
7. Region Server Assignment

