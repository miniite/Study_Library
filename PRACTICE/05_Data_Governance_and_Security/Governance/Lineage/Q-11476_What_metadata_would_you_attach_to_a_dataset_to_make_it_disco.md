---
type: "practice"
difficulty: "Intermediate"
---

# What metadata would you attach to a dataset to make it 'discoverable' and 'trustworthy', and how do you rank search results in a catalog?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Glossary Metadata and Contracts](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Glossary_Metadata_and_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure datasets are both "discoverable" and "trustworthy," we need to attach comprehensive metadata that covers various aspects such as data quality, provenance, compliance, usage policies, and business context. In a catalog, these metadata attributes help in ranking search results based on relevance and reliability.

#### Triage

The question is asking for the metadata necessary to make datasets discoverable (easily findable) and trustworthy (reliable). It also probes into how we can rank search results in a data catalog effectively. Assumptions include that the dataset metadata will be stored in a structured format, such as JSON or XML, and that there are established policies for data governance.

#### First-Principles Solution

Metadata Elements: The key metadata elements to attach to datasets include:

Basic Information: Dataset name, description, version.
Data Quality Metrics: Null values, unique counts, duplicates, etc.
Provenance and Lineage: Source systems, transformation steps, lineage graphs.
Compliance and Governance: Data classification (e.g., PII), regulatory compliance (GDPR, HIPAA).
Usage Policies: Access controls, data usage terms.
Business Context: Relevance to business functions, key performance indicators (KPIs).

Ranking Search Results:

Relevancy Scoring: Higher scores for more relevant datasets based on query keywords and metadata attributes.
Trustworthiness Factors: Higher trust for well-governed data with clear lineage and compliance records.
Detailed Metadata Schema
JSON
{
  "datasetName": "string",
  "description": "string",
  "version": "integer",
  "qualityMetrics": {
    "nullValuesCount": "integer",
    "uniqueCount": "integer",
    "duplicatesFound": "boolean"
  },
  "provenance": [
    {
      "sourceSystem": "string",
      "transformationSteps": ["string"],
      "lineageGraph": "JSON"
    }
  ],
  "compliance": {
    "dataClassification": "string", // e.g., PII, sensitive
    "regulatoryCompliance": ["string"], // e.g., GDPR, HIPAA
    "certifications": ["string"] // e.g., SOC2, ISO27001
  },
  "usagePolicies": {
    "accessControls": {
      "userRoles": ["string"],
      "permissions": ["string"]
    },
    "usageTerms": "string"
  },
  "businessContext": {
    "relevanceToBusinessFunctions": ["string"], // e.g., finance, HR
    "keyPerformanceIndicators": ["string"] // e.g., revenue, customer satisfaction
  }
}
Ranking Search Results

The ranking of search results can be done through a scoring mechanism that considers multiple factors. For instance:

Relevancy: Weighted score based on how well the metadata matches the query keywords.
Trustworthiness: Higher scores for datasets with high-quality lineage, clear compliance records, and established usage policies.
Yes
No
Yes
No
Query
Is Metadata Relevant?
Increase Relevancy Score
Decrease Relevancy Score
Is Data Well-Governed?
Increase Trustworthiness Score
Decrease TrustworthinessScore
Higher Search Rank
Higher Search Rank
Lower Search Rank
Lower Search Rank
Is Dataset Frequently Used?
Is Dataset Old?
Rank Higher, High UsageFrequency
Is Data Recent and Relevant?
Rank Lower, Outdated orIrrelevant
Top of Results List

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, the metadata should cover all necessary aspects to ensure data discoverability and trustworthiness. A robust ranking system can prioritize relevant and reliable datasets based on a combination of relevancy scoring and trustworthiness factors. Common mistakes include overlooking essential metadata elements or not implementing effective ranking mechanisms.

#### Follow-up
questions might explore specific tools for metadata management, such as Apache Atlas or AWS Glue Data Catalog, or discuss strategies for automating the generation and validation of metadata.

**Code:**

```json
{
  "datasetName": "SalesDataQ42023",
  "description": "Quarterly sales data from Q4 2023 including product details, sales amounts.",
  "version": 1,
  "qualityMetrics": {
    "nullValuesCount": 5,
    "uniqueCount": 1500,
    "duplicatesFound": false
  },
  "provenance": [
    {
      "sourceSystem": "ERP",
      "transformationSteps": ["Data cleaning", "Join with marketing data"],
      "lineageGraph": "JSON graph representation"
    }
  ],
  "compliance": {
    "dataClassification": "non-sensitive",
    "regulatoryCompliance": [],
    "certifications": []
  },
  "usagePolicies": {
    "accessControls": {
      "userRoles": ["sales", "finance"],
      "permissions": ["read"]
    },
    "usageTerms": "Data can only be used for internal reporting purposes."
  },
  "businessContext": {
    "relevanceToBusinessFunctions": ["finance", "marketing"],
    "keyPerformanceIndicators": ["total revenue", "customer retention"]
  }
}
```

**Example input / output:**

```text
flowchart TD
    A[User Query] --> B{Is Metadata Relevant?}
    B -->|Yes| C1[Increase Relevancy Score]
    B -->|No| D1[Decrease Relevancy Score]

    A --> E{Is Data Well-Governed?}
    E -->|Yes| C2[Increase Trustworthiness Score]
    E -->|No| D2[Decrease Trustworthiness Score]

    C1 --> F[Higher Search Rank]
    C2 --> G[Higher Search Rank]
    D1 --> H[Lower Search Rank]
    D2 --> I[Lower Search Rank]

    F --> J{Is Dataset Frequently Used?}
    G --> J
    H --> K{Is Dataset Old?}
    I --> K

    J --> L[Rank Higher, High Usage Frequency]
    K --> M{Is Data Recent and Relevant?}
    M --> N[Rank Lower, Outdated or Irrelevant]

    L --> O[Top of Results List]
    M --> N
```

