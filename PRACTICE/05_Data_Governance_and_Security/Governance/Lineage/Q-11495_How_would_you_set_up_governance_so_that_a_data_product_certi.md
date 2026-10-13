---
type: "practice"
difficulty: "Deep"
---

# How would you set up governance so that a data product certified as 'gold' cannot be built from uncertified upstream sources?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Ownership](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a "gold" data product is not derived from uncertified upstream sources, we need to implement a robust governance framework focusing on data lineage and certification mechanisms. The core idea is to establish strict controls over data access and transformation processes, ensuring transparency and traceability of data flow.

#### Triage

The question probes how to enforce strong data governance policies that prevent the use of uncertified data in producing certified ("gold") datasets. This involves understanding current data flows, identifying key transformations, and implementing checks at every step. Assumptions include existing data lineage tools and a commitment to regular audits.

Step-by-Step Solution
Data Lineage Visualization: Utilize tools like Apache Atlas or AWS Glue Data Catalog to map out the entire data pipeline, including all sources, transformations, and sinks.
Certification Tags: Implement tags or labels in your data lineage metadata to mark data as "certified" or "uncertified." These tags should be applied at ingestion and transformation stages.
Transformation Rules: Develop rules that enforce the use of certified upstream data for gold products. This can be implemented using Data Fabric capabilities or custom scripts.
Continuous Monitoring and Auditing: Set up automated checks to verify that only certified data is used in producing gold datasets, with regular manual audits as a backup.
Detailed Implementation

Lineage Visualization:

Use Apache Atlas for metadata management, which provides a centralized repository for lineage information.
Integrate AWS Glue Data Catalog or similar tools to map out the transformation processes and data sources.

Certification Tags:

Define tags such as CERTIFIED, UNCERTIFIED, and possibly intermediate states like PARTIALLY_CERTIFIED.
Apply these tags during data ingestion, processing, and storage using metadata tagging features in your data management tools.

Transformation Rules:

Implement a rule engine or use Data Fabric capabilities to enforce rules that only allow certified data sources for gold products.
For example, a transformation script might check the lineage of the input data before proceeding with any processing steps.

Continuous Monitoring and Auditing:

Use automated tools like Apache Airflow or AWS Glue ETL jobs to monitor data flow and enforce rules.
Set up alerts for when uncertified data is detected in a gold pipeline, triggering manual review processes.

#### Closing Summary

By combining robust data lineage visualization with certification tagging and continuous monitoring, we can ensure that only certified upstream sources are used in producing gold datasets. This approach provides transparency, traceability, and compliance enforcement across the entire data lifecycle.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to implement end-to-end lineage visibility or missing out on intermediate steps where certification could be skipped.

#### Follow-Up
Discuss how to handle edge cases like partially certified datasets and how to manage data that changes over time while maintaining governance compliance.

**Comparison:**

| Transformation | Source Data Certified? | Target Data Certification |
|----------------|------------------------|--------------------------|
| Step 1         | No                     | No                       |
| Step 2         | Yes                    | Yes                      |

**Code:**

```python
# Example Python script for enforcing rule checks during data processing
def process_data(input_data, lineage_info):
    # Check if input data is certified
    if not lineage_info['certified']:
        raise ValueError("Cannot use uncertified data in gold products.")

    # Process the data as usual
    processed_data = transform(input_data)
    return processed_data

# Example lineage info structure
lineage_info = {
    'source': 'uncertified_source',
    'certified': False,
}
```

**Example input / output:**

```
Input:
JSON
Copy
{
  "lineage_info": {
    "source": "s3://raw-data-source",
    "certified": False
  }
}
Output:
PYTHON
Copy
raise ValueError("Cannot use uncertified data in gold products.")
```

**Data:**

```json
{
  "lineage": [
    {
      "source": "s3://raw-data-source",
      "transformations": ["cleaning", "aggregation"],
      "target": "gold-dataset"
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Data Ingestion
4. Is Source Certified?
5. Reject Data
6. Transform and Tag
7. Store Transformed Data

