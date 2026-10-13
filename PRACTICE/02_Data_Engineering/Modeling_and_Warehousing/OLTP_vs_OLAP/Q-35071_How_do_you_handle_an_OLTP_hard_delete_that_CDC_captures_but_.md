---
type: "practice"
difficulty: "Deep"
---

# How do you handle an OLTP hard delete that CDC captures but the OLAP fact table already aggregated — do you tombstone, reverse the fact, or reload the partition, and why?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Feeding OLAP from OLTP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Feeding_OLAP_from_OLTP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling an OLTP hard delete that Change Data Capture (CDC) detects, but the associated data has already been aggregated in an OLAP fact table, we must consider the trade-offs between tombstone records, reversing facts, and reloading partitions. This decision impacts data consistency, performance, storage costs, and the overall integrity of our data warehouse. In this scenario, I would recommend tombstoning the record to preserve historical accuracy without risking data inconsistencies or significantly impacting query performance.

**Code:**

```python
# Example Python code for tombstone handling in a data pipeline

def handle_hard_delete(cdc_record, fact_table):
    # Mark the deleted row as a tombstone
    if cdc_record['action'] == 'delete':
        cdc_record['tombstone'] = True

    return cdc_record
```

**Example input / output:**

```text
Input:
- CDC record: `{"id": 123, "action": "delete"}`
- Fact table state before update: `[{"id": 1, "metric_value": 50}, {"id": 123, "metric_value": 70}]`

Output:
- Updated fact table state: `[{"id": 1, "metric_value": 50}, {"id": 123, "tombstone": true}]`
```

**Data:**

```json
{
  "cdc_record": {
    "id": 123,
    "action": "delete",
    "tombstone": true
  },
  "fact_table": [
    { "id": 1, "metric_value": 50 },
    { "id": 123, "metric_value": null } // tombstone example
  ]
}
```

