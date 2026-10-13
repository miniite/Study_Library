---
type: "practice"
difficulty: "Expert"
---

# You are consolidating five siloed catalogs (Glue, Collibra, Alation, an internal wiki, and Confluence pages) into one org-wide metadata platform. Walk me through your target-state architecture and how you decide what becomes the system of record for each metadata type.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cataloging External Data](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Cataloging_External_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR:

When consolidating multiple siloed data catalogs into an org-wide metadata platform, the key is to ensure a unified view while respecting the strengths and limitations of existing systems. We will design a hybrid architecture that integrates these sources and defines clear roles for each system as a system of record (SoR). Specifically, we'll make Glue the SoR for AWS metadata, Collibra for enterprise-level metadata, Alation for data discovery, and internal wiki/Confluence pages as secondary references.

#### Triaging the Question

The question is essentially about architecting a unified metadata platform by integrating multiple siloed sources. The interviewer is probing your understanding of how to choose between different systems as the single source of truth (SoR) and your ability to design an efficient, scalable architecture that leverages the strengths of each system.

#### Assumptions
Data Quality: All data catalogs have varying degrees of data quality.
Ownership and Maintenance: There are existing stakeholders responsible for maintaining each catalog.
Regulatory Compliance: The platform must comply with any organizational or regulatory requirements.
User Access Patterns: Different user roles will have varying access patterns to the metadata.

#### First-Principles Solution

Identify Commonalities and Differences:

Evaluate the common data entities and metadata types across all catalogs (e.g., tables, columns, descriptions).
Identify unique strengths of each system (e.g., Glue for AWS resources, Collibra for governance).

Define Systems of Record (SoR):

For AWS-specific metadata, choose Glue as the SoR.
For enterprise-level and regulatory compliance metadata, select Collibra as the primary SoR.
Use Alation as a reference for data discovery capabilities.
Internal wiki/Confluence pages will be maintained but not the SoR.

Design Integration Strategy:

Develop ETL processes to sync Glue’s metadata with Collibra and other catalogs.
Implement API integrations where possible to automate updates between systems.
Use a data lake or central repository as an intermediary layer for consistent data storage.

Architectural Components:

Metadata Hub: A central database storing unified metadata.
Data Lake: Storage layer for raw and processed metadata.
API Gateway: Facilitates secure access to the metadata hub and other systems.
Monitoring & Alerting: Real-time monitoring of data freshness and integrity.

Security Considerations:

Implement role-based access controls (RBAC) based on user roles.
Use encryption for sensitive metadata during transit and at rest.
Ensure compliance with organizational policies, such as GDPR or HIPAA if applicable.

User Interface & Experience:

Design a unified UI that integrates insights from multiple sources.
Provide search capabilities across different catalogs.
Enable advanced analytics using tools like Jupyter Notebooks for data scientists.

#### Closing Summary

By carefully defining the SoR and integrating various systems, we can create a robust, scalable metadata platform. The architecture ensures consistency while leveraging the strengths of each system. Regular audits and updates will be crucial to maintain accuracy and relevance over time.

#### Common Mistakes/Follow-Ups
Mistake: Failing to define clear roles for each SoR or integrating too many systems without a strategy.

#### Follow-Up
How do you handle data inconsistencies between sources?

#### Follow-Up
What are the performance implications of your chosen architecture?

**Comparison:**

```
Table	Columns	Description
users	user_id	Unique identifier
	username	User's name
	email	User's contact email
```

**Code:**

```python
# Example ETL Job in Glue to sync metadata with Collibra
def sync_metadata_with_collibra(glue_table, collibra_url):
    # Fetch metadata from Glue
    glue_data = fetch_glue_metadata(glue_table)

    # Prepare data for Collibra API
    collibra_payload = prepare_payload_for_collibra(glue_data)

    # Send to Collibra API
    response = requests.post(f"{collibra_url}/api/metadata", json=collibra_payload)
    if response.status_code == 200:
        print("Sync successful")
    else:
        print("Error syncing metadata: ", response.text)
```

**Example input / output:**

```text
Input:
- Sync metadata from Glue table 'users' to Collibra

Output:
- Metadata for table 'users' successfully synced with Collibra.
- Updated metadata stored in Collibra API response.
```

**Data:**

```json
{
  "glue_table": {
    "name": "users",
    "columns": ["user_id", "username", "email"],
    "description": "User information"
  },
  "collibra_metadata": [
    {
      "table_name": "users",
      "column": "user_id",
      "description": "Unique identifier for the user"
    }
  ]
}
```

**Flow:**

1. Glue Metadata
2. Data Lake
3. Metadata Hub
4. Collibra API
5. API Gateway
6. User Interface

