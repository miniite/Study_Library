---
type: "practice"
difficulty: "Expert"
---

# You inherit a data swamp: an S3 feature lake, no catalog, no ownership, 5PB of unknown provenance feeding live models. Give the 90-day and 12-month plan to govern it without freezing any model's retraining.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse Vendor Exit Strategy](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_Vendor_Exit_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

Inheriting a sprawling data swamp with an S3 feature lake, no catalog, and unknown provenance presents significant challenges but also opportunities to establish robust governance while minimizing disruption. The 90-day plan focuses on understanding the current state, defining a minimal ownership model, and setting up basic metadata. The 12-month plan aims for comprehensive cataloging, automated data lineage tracking, and alignment with broader organizational policies.

#### Triaging the Question

The interviewer is probing your ability to handle large-scale legacy systems and implement effective governance structures without disrupting ongoing operations. Key points include:

Understanding current state: What does "unknown provenance" mean? Are there any existing models or processes?
Ownership model: Who will be responsible for data, metadata, and model updates?
Initial steps vs long-term vision: How do you balance immediate needs with future scalability?

#### Assumptions
The S3 lake contains a variety of feature sets used by multiple models.
There are existing live models using some of these features.
Some form of monitoring or alerting is in place for model performance.
90-Day Plan:
Step 1: Initial Assessment
Objective: Understand the current state and dependencies.
Action: Use AWS Athena or Glue to query S3 to discover available feature sets, their size, and recent access patterns. Identify which models are currently using these features.
Initial S3 Data Scan
Discovered Feature Sets
Identify Live Models UsingFeatures
Step 2: Define Minimal Ownership Model
Objective: Establish clear roles and responsibilities for managing data, metadata, and model updates.
Action: Create a small ownership team consisting of key stakeholders (data engineers, ML engineers, data scientists). Define minimal SLAs for each role.
MARKDOWN
| Role | Responsibilities |
|------|------------------|
| Data Engineer | Maintain feature sets, ensure data quality, support querying tools. |
| Data Scientist | Ensure model accuracy using up-to-date features. |
| Model Owner | Oversee the use of specific models and their dependencies. |
Step 3: Set Up Basic Metadata
Objective: Create a lightweight metadata catalog to track basic information about each feature set.
Action: Use AWS Glue Data Catalog or custom-built metadata store to start storing basic metadata such as feature names, data types, last updated timestamps.
Basic Metadata Setup
Metadata Store
Feature Set Information
12-Month Plan:
Step 4: Comprehensive Cataloging
Objective: Create a comprehensive metadata catalog with detailed lineage tracking.
Action: Integrate AWS Glue Data Catalog fully, ensuring all feature sets are registered. Use automated tagging and labeling to enrich metadata.
Comprehensive MetadataCatalog
Automated Tagging
Lineage Tracking
Step 5: Automated Data Lineage
Objective: Track data lineage automatically to understand how features are used in models.
Action: Integrate AWS Glue Data Catalog with ML workflows, using automated tagging and lineage discovery tools.
Automated Data Lineage
Data Lake Integration
Model DependencyManagement
Step 6: Compliance and Policy Alignment
Objective: Ensure all data management practices align with organizational policies.
Action: Work with compliance teams to establish data governance policies. Use AWS Config or similar tools to enforce these policies.
Compliance Policies
Policies Enforcement
Model Retraining Governance

#### Closing Summary

By following the 90-day and 12-month plans, we will establish a robust governance framework for the data lake while minimizing disruptions to live models. The key is to start with minimal effort but build towards comprehensive solutions over time.

#### Common Mistakes/Follow-Ups
Over-reliance on upfront modeling can freeze model retraining.
Lack of clear ownership and roles can lead to operational bottlenecks.
Failing to integrate data lineage early can complicate troubleshooting later.
Not aligning with broader organizational policies may result in compliance issues.

**Comparison:**

```
Feature Set	Data Type	Last Updated
set1	JSON	2023-04-01
set2	CSV	2023-05-01
```

**Code:**

```python
# Example Athena query for initial S3 scan
query = """
SELECT
    s3_bucket,
    s3_key,
    size_bytes,
    last_modified_date
FROM
    S3Objects
"""
```

**Example input / output:**

```
Input S3 Scan	Output Identified Features
SELECT ...	{"featuresets": ["set1", "set2"], "models_using_features": ["modelA", "modelB"]}
```

**Data:**

```yaml
data:
  - s3_bucket: my-feature-lake
  - s3_key: features/2023/q1/featuresets/*
  - size_bytes: 500GB
  - last_modified_date: "2023-04-01"
```

**Flow:**

1. Initial S3 Scan
2. Identify Features
3. Register Metadata
4. Tagging and Labeling
5. Automated Lineage Tracking

