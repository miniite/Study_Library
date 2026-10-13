---
type: "practice"
difficulty: "Expert"
---

# Design how tags/classifications propagate through lineage automatically — if an upstream column is tagged PII, downstream derived columns should inherit it. How do you prevent over-propagation (everything becomes PII) and under-propagation?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Lineage Graphs](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Scaling_Lineage_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system where tags like Personally Identifiable Information (PII) propagate through data lineage automatically, we need to ensure both accurate tagging and efficient propagation without over- or under-tagging. The core idea is to implement a robust metadata management framework that tracks lineage and applies appropriate logic to tag downstream columns based on upstream dependencies.

The key steps are:

Define Tag Rules: Establish clear rules for how tags should propagate, such as if an upstream column is tagged PII, any derived column must inherit this tag.
Lineage Tracking: Maintain a detailed lineage graph that captures the relationships between data sources and transformations.
Tag Inheritance Logic: Implement algorithms to apply tags based on lineage rules without over- or under-tagging.

To prevent over-propagation (everything becomes PII) and under-propagation, we must carefully design our logic and use appropriate heuristics.

**Comparison:**

```
Transformation Rule	Upstream Column	Downstream Columns	Tag
Hash	customer_id	customer_name	PII
Hash	customer_id	customer_address	PII
Hash	email	email_hash	PII
```

**Code:**

```python
# Example Python function for tag propagation
def propagate_tag(upstream_column_tags, transformation_rules):
    # Initialize the output dictionary with default values
    downstream_tags = {column: None for column in transformed_columns}

    # Apply transformation rules to determine tags for each downstream column
    for upstream_column, transformations in transformation_rules.items():
        if 'PII' in upstream_column_tags.get(upstream_column, []):
            for downstream_column, transform_type in transformations:
                downstream_tags[downstream_column] = 'PII'

    return downstream_tags

# Example usage
upstream_column_tags = {'customer_id': ['PII'], 'email': ['PII']}
transformation_rules = {
    'customer_id': [('customer_name', 'hash'), ('customer_address', 'hash')],
    'email': [('email_hash', 'hash')]
}
result = propagate_tag(upstream_column_tags, transformation_rules)
print(result)
```

**Example input / output:**

```text
Input:
- upstream_column_tags = {'customer_id': ['PII'], 'email': ['PII']}
- transformation_rules = {
  'customer_id': [['customer_name', 'hash'], ['customer_address', 'hash']],
  'email': [['email_hash', 'hash']]
}

Output:
- downstream_tags = {'customer_name': 'PII', 'customer_address': 'PII', 'email_hash': 'PII'}
```

**Data:**

```json
{
  "upstream_column_tags": {
    "customer_id": ["PII"],
    "email": ["PII"]
  },
  "transformation_rules": {
    "customer_id": [
      ["customer_name", "hash"],
      ["customer_address", "hash"]
    ],
    "email": [["email_hash", "hash"]]
  }
}
```

**Flow:**

1. Yes
2. No
3. Transformations
4. customer_id
5. Transform to customer_name
6. customer_name
7. customer_id
8. Transform tocustomer_address
9. customer_address
10. email
11. Transform to email_hash
12. email_hash
13. Upstream Column
14. Is tagged PII?
15. Tag Downstream Columns
16. No Tag Change

