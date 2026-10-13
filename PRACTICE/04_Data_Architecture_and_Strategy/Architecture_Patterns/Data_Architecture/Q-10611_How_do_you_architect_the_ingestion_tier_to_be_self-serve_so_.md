---
type: "practice"
difficulty: "Expert"
---

# How do you architect the ingestion tier to be self-serve so domains onboard new sources without the platform team, while keeping schema registration, PII classification, and cataloging automatic and mandatory?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Proving PII Residency](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architected correctly, the ingestion tier can become a self-serve mechanism that allows domain teams to onboard new data sources without involving the platform team. This involves designing an automated schema registration process, PII classification system, and cataloging framework to ensure consistency and compliance. The key is to balance ease of use with robustness and security.

To achieve this, we need to address several aspects:

Self-Serve Ingestion: Ensure domain teams can easily configure and start ingesting new data sources.
Schema Registration Automation: Automatically capture and document the schema of incoming data without manual intervention.
PII Classification: Implement a system that automatically identifies and labels Personally Identifiable Information (PII) to enforce strict privacy policies.
Cataloging: Create an inventory of all ingested data sources, their schemas, and metadata.

Let's break this down step by step using first-principles thinking.

**Comparison:**

| Schema | Fields |
|--------|--------|
| sales  | transaction_id, customer_name, amount |

**Code:**

```python
# Example schema registration script in Python
def register_schema(schema):
    # Save the schema to a centralized storage (e.g., JSON)
    with open('schemas.json', 'a') as f:
        json.dump(schema, f)
        f.write('\n')
```

**Example input / output:**

```text
Input (schema registration request):
{
  "source": "sales",
  "schema": {
    "fields": [
      {"name": "transaction_id", "type": "string"},
      {"name": "customer_name", "type": "string"},
      {"name": "amount", "type": "float"}
    ]
  }
}

Output (registered schema):
{
  "source": "sales",
  "schema": {
    "fields": [
      {"name": "transaction_id", "type": "string"},
      {"name": "customer_name", "type": "string"},
      {"name": "amount", "type": "float"}
    ],
    "timestamp": "2023-10-01T14:48:00Z"
  }
}
```

**Data:**

```json
{
  "source": "sales",
  "schema": {
    "fields": [
      {"name": "transaction_id", "type": "string"},
      {"name": "customer_name", "type": "string"},
      {"name": "amount", "type": "float"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Domain Team uploads schema
4. Is the schema valid?
5. Save to central storage
6. Request corrections fromdomain team
7. Notify platform team forvalidation
8. Validation successful
9. Catalog updated automatically
10. Validation failed
11. Correct schema and resubmit

