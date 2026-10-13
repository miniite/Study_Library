---
type: "practice"
difficulty: "Deep"
---

# What's your strategy for governing a single canonical identity resolution so every retrieved fact about a customer maps to one entity across all data sources?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Policy as Code for Data](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Policy_as_Code_for_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern a single canonical identity resolution, ensuring that every retrieved fact about a customer maps to one entity across all data sources, we need to focus on establishing robust data governance frameworks, implementing advanced identity resolution techniques, and fostering a culture of data stewardship. This strategy involves multiple layers including data cataloging, semantic consistency, real-time reconciliation, and continuous monitoring.

The core idea is that by having a single, unified view of the customer—referred to as a "golden record"—we can ensure that all data about a specific customer is consistent, accurate, and up-to-date. This golden record acts as the central hub for integrating disparate sources while maintaining identity integrity.

To achieve this, we need to:

Define the Identity Model: Clearly define what constitutes an identity across different systems.
Implement Data Cataloging: Create a comprehensive data catalog that documents all relevant customer data from various sources.
Use Advanced Matching Algorithms: Leverage techniques like fuzzy matching and probabilistic matching to resolve identities accurately.
Establish a Real-Time Reconciliation System: Ensure that changes in one system are reflected across others in real-time.
Maintain Data Quality: Implement continuous quality checks and corrective actions.

**Comparison:**

```
Data Source	Customer ID	Name
Source A	456	Alice Smith
Source B	789	Alice Smithe
Source C	1012	Alyce Smith
```

**Code:**

```python
# Example of fuzzy matching using Levenshtein distance for identity resolution
from fuzzywuzzy import fuzz

def resolve_customer_id(customer_name, customer_db):
    max_similarity = 0
    best_match = None
    for db_entry in customer_db:
        similarity_score = fuzz.ratio(customer_name, db_entry['name'])
        if similarity_score > max_similarity:
            max_similarity = similarity_score
            best_match = db_entry['id']
    return best_match

# Example usage
customer_db = [
    {'id': 1, 'name': 'Alice Smith'},
    {'id': 2, 'name': 'Alice Smithe'},
    {'id': 3, 'name': 'Alyce Smith'}
]
best_id = resolve_customer_id('Alice Smith', customer_db)
print(best_id)  # Output: 1 or 2 based on the highest similarity
```

**Example input / output:**

```text
Input:
- Customer name: Alice Smith

Output:
- Best match ID: 1 or 2 (based on the highest similarity score using fuzzy matching)
```

**Data:**

```json
{
    "customerDatabase": [
        {"id": 1, "name": "Alice Smith"},
        {"id": 2, "name": "Alice Smithe"},
        {"id": 3, "name": "Alyce Smith"}
    ]
}
```

**Flow:**

1. Data Ingestion
2. Data Cataloging
3. Identity Resolution
4. Golden Record Storage
5. Data Reconciliation
6. Monitoring & Maintenance

