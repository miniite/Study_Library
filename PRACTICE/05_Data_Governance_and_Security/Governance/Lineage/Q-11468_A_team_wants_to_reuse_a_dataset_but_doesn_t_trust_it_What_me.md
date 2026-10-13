---
type: "practice"
difficulty: "Intermediate"
---

# A team wants to reuse a dataset but doesn't trust it. What metadata (freshness, quality, ownership, lineage) would you surface to build trust?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Glossary Metadata and Contracts](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Glossary_Metadata_and_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the trust issue with a reused dataset, we need to ensure that all relevant metadata is surfaced and understood by the team. Metadata includes freshness (how recent the data is), quality (data integrity measures such as completeness, accuracy, and consistency), ownership (who is responsible for maintaining the data), and lineage (history of data transformations). By systematically gathering and presenting this information, we can build a robust framework that enhances trust in the dataset.

#### Triaging the Question

The question asks us to identify the specific metadata elements necessary to establish trust when reusing a potentially untrusted dataset. The key areas to probe are:

Freshness: Ensuring the data is current and relevant.
Quality: Checking for any inherent issues that could affect analysis.
Ownership: Identifying who is responsible for maintaining the integrity of the data.
Lineage: Understanding the history and transformations applied to the data.

#### First Principles Solution

Freshness:

Timestamps: Include start time, end time, and processing timestamp in the metadata.
Frequency of updates: Specify how often the dataset is refreshed.

Quality:

Validation rules: Define any checks that ensure data integrity (e.g., non-null values, valid formats).
Error rates: Report on any anomalies or errors found during recent processes.
Data completeness metrics: Provide measures like null value counts or missing records.

Ownership:

Data stewards: Identify the individuals responsible for maintaining and validating the data.
Contact information: Ensure there is a point of contact for questions and issues.

Lineage:

Transformation details: Document each step in the data processing pipeline, including transformations applied and tools used.
Dependencies: List any upstream datasets or external systems that influence this dataset.
Historical changes: Maintain a log of significant changes to the dataset over time.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By surfacing these metadata elements, we can provide comprehensive information about the data's state and history. This approach helps build trust by allowing stakeholders to verify the data’s quality and relevance. However, some common mistakes include:

Omitting critical details: Not including all necessary validation rules or transformation steps.
Lack of up-to-date information: Failing to update metadata regularly can lead to misleading reports.

#### Follow-ups
might involve discussing how often these metadata updates are performed and what tools are used for tracking lineage.

**Comparison:**

| Metadata Element | Details                                                                                         |
|------------------|-------------------------------------------------------------------------------------------------|
| Dataset Name     | CustomerActivity                                                                                |
| Freshness        | Last Update: 2023-10-05 14:30; Processing Time: daily                                           |
| Quality          | Null Values: 12; Data Type Errors: 5                                                            |
| Ownership        | Data Stewards: John Doe, Jane Smith; Contact Email: data.steward@example.com                    |

**Code:**

```python
# Example code snippet for generating a metadata report

def generate_metadata_report(dataset_name, freshness_info, quality_check_results, ownership_details):
    report = {
        "dataset_name": dataset_name,
        "freshness": freshness_info,
        "quality": quality_check_results,
        "ownership": ownership_details
    }

    return report

# Example usage
metadata_report = generate_metadata_report(
    dataset_name="CustomerActivity",
    freshness_info={"last_update": "2023-10-05 14:30", "processing_time": "daily"},
    quality_check_results={"null_values": 12, "data_type_errors": 5},
    ownership_details={"data_stewards": ["John Doe", "Jane Smith"], "contact_email": "data.steward@example.com"}
)
print(metadata_report)
```

**Example input / output:**

```text
Metadata Report:
- Dataset Name: CustomerActivity
- Freshness:
  - Last Update: 2023-10-05 14:30
  - Processing Time: daily
- Quality:
  - Null Values: 12
  - Data Type Errors: 5
- Ownership:
  - Data Stewards: John Doe, Jane Smith
  - Contact Email: data.steward@example.com
```

**Data:**

```json
{
  "dataset_name": "CustomerActivity",
  "freshness": {
    "last_update": "2023-10-05 14:30",
    "processing_time": "daily"
  },
  "quality": {
    "null_values": 12,
    "data_type_errors": 5
  },
  "ownership": {
    "data_stewards": ["John Doe", "Jane Smith"],
    "contact_email": "data.steward@example.com"
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Generate Freshness Report] --> B{Is Dataset Updated?};
4. B -- Yes --> C[Record Update Timestamp];
5. B -- No --> D{Is Processing Time Known?};
6. D -- Yes --> E[Record Processing Time];
7. D -- No --> F[Skip Record Processing Time];
8. C OR F --> G[Generate Quality Check Report];
9. G --> H{Are There Any Errors?};
10. H -- Yes --> I[Log Error Details];
11. H -- No --> J[Generate Ownership Details];
12. I OR J --> K[Generate Lineage Details];
13. K --> L[Compile All Metadata into Final Report];

