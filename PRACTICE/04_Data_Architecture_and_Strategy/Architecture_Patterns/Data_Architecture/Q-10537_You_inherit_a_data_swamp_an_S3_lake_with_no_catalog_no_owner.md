---
type: "practice"
difficulty: "Expert"
---

# You inherit a data swamp: an S3 lake with no catalog, no ownership, and 3PB of unknown provenance. Give me the 90-day and 12-month plan to make it governed without freezing every downstream consumer.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse Vendor Exit Strategy](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_Vendor_Exit_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Inheriting a data swamp with an unstructured S3 lake requires a strategic, phased approach to govern the data without disrupting existing consumers. The 90-day plan focuses on setting up basic metadata and cataloging tools, while the 12-month plan involves implementing a comprehensive governance framework and integrating it seamlessly into the existing ecosystem.

#### Triaging the Question

The interviewer is probing your ability to design a robust data governance strategy in a challenging environment. They are likely interested in understanding how you would approach setting up metadata, establishing ownership, ensuring data quality, and maintaining ongoing maintenance without causing disruption to downstream consumers.

#### Assumptions
Downstream Consumers: Assume there are existing consumers using the S3 lake for various purposes.
Team Resources: The team has access to DevOps tools and engineers.
Tools & Technologies: Familiarity with AWS services, metadata cataloging tools like Glue or Delta Lake, and data quality tools.

#### First-Principles Solution
90-Day Plan

Initial Inventory and Assessment

Task: Use AWS Athena to run queries on the S3 lake to understand the structure and volume of data.
Tool: Write a script that automates the extraction of metadata such as file paths, sizes, and last modified dates.

Metadata Catalog Setup

Task: Deploy AWS Glue Data Catalog or similar tools to start cataloging the metadata.
Tool: Use AWS Glue Crawler to automatically discover and catalog the data stored in S3.

Establish Ownership and Governance Teams

Task: Form a governance team responsible for defining policies, standards, and procedures.
Task: Assign ownership of different datasets based on their business relevance and criticality.

Data Quality Checks

Task: Implement basic data quality checks using AWS Glue or other ETL tools.
Tool: Develop a set of common data validation rules that can be applied across the lake.
12-Month Plan

Enhance Metadata and Cataloging

Task: Integrate metadata with existing analytics tools for better visibility.
Tool: Use AWS Glue Data Catalog to enrich metadata with additional attributes like lineage, schema evolution history.

Implement Advanced Governance Policies

Task: Develop a comprehensive set of data governance policies covering security, privacy, and regulatory compliance.
Task: Automate policy enforcement using AWS Lake Formation or custom solutions.

Optimize Data Storage and Retrieval

Task: Optimize S3 storage classes based on data access patterns.
Tool: Use AWS S3 Intelligent-Tiering to automatically manage the storage tiers.

Integrate with Existing Ecosystems

Task: Ensure seamless integration of new governance features with existing downstream consumers.
Task: Conduct regular meetings with stakeholders to gather feedback and ensure smooth adoption.

Continuous Monitoring and Improvement

Task: Set up continuous monitoring for data quality, access patterns, and compliance.
Tool: Use AWS CloudWatch or custom dashboards to monitor key metrics.

#### Closing Summary

The 90-day plan sets the foundation by cataloging existing data assets, establishing governance frameworks, and implementing basic data quality checks. The 12-month plan builds on this foundation, enhancing metadata management, enforcing advanced policies, and optimizing storage and retrieval processes while ensuring minimal disruption to downstream consumers.

#### Common Mistakes/Follow-Ups
Freezing Downstream Consumers: Avoid locking down the lake too early; ensure gradual integration.
Overcomplicating Governance: Start with basic but effective governance principles and build complexity as needed.
Neglecting Data Quality: Poor data quality can undermine trust in the governed data, so continuous monitoring is crucial.

**Comparison:**

```
Data Quality Rule	Description
Valid Dates	Ensure all dates are within valid range
Unique Keys	Check for duplicate primary keys
Column Lengths	Verify column lengths match expectations
Missing Values	Identify records with missing values
```

**Code:**

```python
# Example Python script for metadata inventory using AWS Athena
import boto3

s3 = boto3.client('s3')
athena = boto3.client('athena')

def get_s3_objects(bucket):
    paginator = s3.get_paginator('list_objects_v2')
    response_iterator = paginator.paginate(Bucket=bucket)

    for page in response_iterator:
        if 'Contents' in page:
            for obj in page['Contents']:
                print(obj['Key'])

get_s3_objects('your-s3-bucket-name')

# Example AWS Glue Crawler setup
def start_glue_crawler():
    glue = boto3.client('glue')

    response = glue.start_crawler(
        Name='your-glue-crawler-name'
    )

    print(response)

start_glue_crawler()
```

**Example input / output:**

| Input (S3 Object Key) | Output (Metadata) |
|-----------------------|--------------------|
| data/2021/q4/sales.csv | {key: "data/2021/q4/sales.csv", size: 51200, last_modified: "2022-02-01T12:00:00Z"} |

**Data:**

```json
{
  "s3_objects": [
    {
      "key": "data/2021/q4/sales.csv",
      "size": 51200,
      "last_modified": "2022-02-01T12:00:00Z"
    },
    ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Start
6. Is S3 Lake Empty?
7. Initialize Metadata Inventory
8. Use Existing Metadata
9. Set Up AWS Glue Crawler
10. Establish Ownership andGovernance Teams
11. Implement Basic Data QualityChecks
12. End of 90-Day Plan
13. Start
14. Is Metadata Complete?
15. Enhance Metadata andCataloging
16. Develop AdvancedGovernance Policies
17. Integrate with ExistingEcosystems
18. Continuous Monitoring andImprovement
19. End of 12-Month Plan

