---
type: "practice"
difficulty: "Expert"
---

# You must set data-governance and compliance architecture for a search platform holding PII across GDPR, CCPA, and India's DPDP jurisdictions. How do you architect right-to-be-forgotten, data residency, and field-level access without crippling query performance?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Platform Scaling](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Platform_Scaling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

To architect a search platform that complies with GDPR, CCPA, and India's DPDP while ensuring right-to-be-forgotten (RTBF) capabilities, data residency control, and field-level access controls without sacrificing query performance, we need to design an elastic architecture that balances security, compliance, and performance. The solution involves using Elasticsearch, alongside a robust governance layer for managing PII.

#### Triage

The core of the question revolves around designing a search platform compliant with multiple data protection regulations while ensuring:

Right-to-be-forgotten: Users can request their data to be deleted from the system.
Data Residency: Data must reside in specific geographic regions as per regulatory requirements.
Field-Level Access Controls: Fine-grained access control over PII fields.

#### First-Principles Solution

Multi-Tier Architecture:

Data Ingestion Tier: Use Elasticsearch for indexing and storing data with appropriate security measures.
Governance Tier: Implement a separate layer to manage PII policies, user requests, and enforcement mechanisms.

Elasticsearch Configuration:

Indexing Strategy: Create indices per jurisdiction and region to enforce data residency.
Security Plugins: Use native Elasticsearch plugins like X-Pack for role-based access control (RBAC) and audit logs.

Data Residency Compliance:

Regional Clusters: Deploy Elasticsearch clusters in different regions, each storing data relevant to that region’s regulatory requirements.
Replication Strategy: Configure cross-cluster replication or sharding strategies to ensure data remains within the required geographic boundaries.

Right-to-be-Forgotten Implementation:

User Request Handling: Implement a centralized request handling mechanism using APIs.
Data Deletion: Use Elasticsearch’s _delete_by_query API to remove PII from relevant indices.
Audit Trails: Maintain logs of all delete requests and actions taken.

Field-Level Access Control:

Dynamic Indexing Patterns: Use dynamic templates in Elasticsearch for field-level encryption and masking.
RBAC Rules: Define and enforce RBAC rules based on user roles and jurisdiction-specific access policies.

#### Example
Architecture
Data Ingestion Tier
Elasticsearch Cluster 1
Region: EU
Elasticsearch Indices
Region: US
Elasticsearch Indices
Region: India
Elasticsearch Indices
Governance Tier
PII Policies Management API
Audit Logs Database
RTBF Request Handling Logic
RTBF Request Handling Logic
RTBF Request Handling Logic

#### Closing Summary
and Common Mistakes

To summarize, the key is to design a multi-tier architecture that leverages Elasticsearch for storage while implementing a governance layer for compliance. Common mistakes include:

Overlooking regional data residency requirements.
Underestimating the complexity of managing multiple PII policies.
Not having robust logging and audit mechanisms.
Final Thoughts

By carefully designing this architecture, we can ensure high query performance while adhering to stringent regulatory requirements.

**Comparison:**

```
Index	Region	PII Policies
user_data_eu	EU	GDPR, CCPA
user_data_us	US	CCPA
user_data_in	IN	DPDP
```

**Code:**

```yaml
# Example Elasticsearch Index Template for Field-Level Access Control
PUT /_template/pii_field_template
{
  "index_patterns": ["pii_data*"],
  "settings": {
    "analysis": {
      "analyzer": {
        "masked_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": ["remove_masked_chars"]
        }
      }
    },
    "max_result_window": 10000
  },
  "mappings": {
    "properties": {
      "masked_field": {
        "type": "text",
        "analyzer": "masked_analyzer"
      }
    }
  },
  "order": 0
}
```

**Example input / output:**

```text
Input:
POST /user_data_eu/_delete_by_query
{
  "query": {
    "bool": {
      "must": [
        { "match": { "email": "example@domain.com" } },
        { "term": { "region": "EU" } }
      ]
    }
  }
}

Output:
{"took":102,"timed_out":false,"_shards":{"total":3,"successful":3,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":null,"hits":[]}}
```

**Data:**

```json
{
  "index": "user_data_eu",
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request
6. Is RTBF Request?
7. Delete PII from Elasticsearch
8. Access Control Check
9. Is User Authorized?
10. Query Elasticsearch
11. Deny Access
12. Audit Log Entry

