---
type: "practice"
difficulty: "Expert"
---

# How do you sequence a migration from a public REST API v1 to v2 where v2 changes core resource modeling? Discuss dual-running both versions, translation between them, cost of maintaining two, and how you avoid v2 becoming "v1 with the same bad decisions."

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Versioning and Deprecation Governance](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Versioning_and_Deprecation_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating from a public REST API v1 to v2 where core resource modeling changes, the key is to ensure a smooth transition without breaking existing functionality and user trust. The strategy involves dual-running both versions, implementing translation layers, carefully managing costs, and ensuring that v2 avoids the pitfalls of v1.

The primary goal is to maintain compatibility with existing clients while gradually introducing new features and improvements in v2. This approach helps mitigate risks and ensures a seamless transition for all stakeholders involved.

**Comparison:**

```
v1_field	v2_field
id	id
name	title
description	summary
```

**Code:**

```python
# Example code to illustrate translation logic between API versions
def translate_v1_to_v2(v1_data):
    # Mapping from v1 fields to v2 fields
    mapping = {
        "v1_field_a": "v2_field_1",
        "v1_field_b": "v2_field_2"
    }

    v2_data = {}
    for v1_key, v2_key in mapping.items():
        if v1_key in v1_data:
            v2_data[v2_key] = v1_data[v1_key]

    # Additional transformations
    v2_data["new_v2_field"] = "default_value"  # Add new fields as needed

    return v2_data
```

**Example input / output:**

```text
Input v1_data:
{
    "id": 123,
    "name": "example_name",
    "description": "v1 description"
}

Output v2_data after translation:
{
    "id": 123,
    "title": "example_name",  # Translated from 'name'
    "summary": "v1 description"  # Translated and renamed to 'summary'
}
```

**Data:**

```json
{
    "v1_resource": {
        "id": 123,
        "name": "example_name",
        "description": "v1 description"
    },
    "v2_resource": {
        "id": 123,
        "title": "example_title",
        "summary": "v2 summary"
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User request
6. Is API version v1?
7. Translate to v2
8. Return translated data
9. Directly return v2 data
10. Check for resource updates inv2
11. Update v2 database and notifyusers
12. End request processing

