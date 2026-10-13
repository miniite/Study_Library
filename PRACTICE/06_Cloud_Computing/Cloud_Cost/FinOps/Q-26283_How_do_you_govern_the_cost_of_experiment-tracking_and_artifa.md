---
type: "practice"
difficulty: "Deep"
---

# How do you govern the cost of experiment-tracking and artifact storage (MLflow/W&B runs, checkpoints, tensorboards) that accumulate forever because engineers fear deleting a run they might need? Design the lifecycle and cost-accountability that curbs the sprawl without losing lineage.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Retention Costs and Compliance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Retention_Costs_and_Compliance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern the cost of experiment-tracking and artifact storage in ML workflows, such as those using MLflow or Weights & Biases (W&B), while maintaining necessary lineage and preventing uncontrolled sprawl, we need to design a comprehensive lifecycle management system with strong cost-accountability. This involves identifying when experiments can be archived, defining clear ownership and access policies, and implementing automated cleanup processes.

#### Triaging the Question

The core question here is about balancing the long-term value of keeping ML experiment records against the financial costs associated with storing these data indefinitely. The interviewer likely wants to explore your understanding of lifecycle management, cost governance, and how to ensure that historical lineage is preserved while minimizing unnecessary storage costs. They may also be interested in hearing about your approach to defining and enforcing policies.

#### Assumptions
Experiment Data Growth: Experiments and their artifacts grow over time.
Cost Concerns: There are significant financial implications of storing data indefinitely.
Lineage Preservation: Keeping historical records is crucial for reproducibility and auditing purposes.
Policy Enforcement: There should be a systematic way to enforce cost controls without manual intervention.

#### First-Principles Solution
1. Define the Experiment Lifecycle Phases

Initial Phase:

Active Experiments: These are currently being run or tested by engineers.
Stable Experiments: Models that have passed initial testing and validation, but may still be under active development.

Archival Phase:

Deprecated Experiments: Models that have been replaced or deemed obsolete due to better performance metrics.
Historical Experiments: Old models and their data that are no longer actively used for training but need to be retained for auditing purposes.
2. Establish Clear Ownership and Access Policies
Ownership: Assign clear ownership of experiments to specific teams or individuals.
Access Control: Implement role-based access control (RBAC) to ensure only authorized personnel can access archived data.
3. Automated Cleanup Strategies
Retention Periods: Define a retention period for different types of experiments, e.g., active/experiment phases might have shorter retention periods while historical phases could be longer.
Automated Deletion: Use cloud-native tools and scripts to automatically delete or archive old experiment data based on predefined policies.
4. Cost Monitoring and Alerts
Budget Allocation: Allocate a budget for ML experiment storage.
Cost Alerts: Set up alerts to notify stakeholders when costs exceed the allocated budget.
Cost Optimization: Regularly review cost metrics and suggest optimization strategies, such as using cheaper storage tiers or implementing data compression.
5. Lineage Preservation
Data Versioning: Implement version control for experiments and artifacts to maintain lineage even after deletion of redundant runs.
Metadata Management: Ensure metadata is preserved in a structured way so that lineage can be reconstructed if necessary.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key steps involve defining clear lifecycle phases, establishing strong ownership and access policies, implementing automated cleanup processes, setting up robust cost monitoring, and ensuring lineage preservation. Common mistakes include neglecting cost governance, failing to enforce policies consistently, and not maintaining adequate lineage.

#### Follow-ups
might explore specific tools and technologies for implementation (e.g., AWS Glue, Google Cloud Dataflow), detailed examples of policy enforcement, or strategies for handling edge cases where engineers might request exceptions from the standard lifecycle rules.

**Comparison:**

```
Experiment	Creation Date	Status
Exp-1	2023-01-01 08:00	Active
Exp-2	2023-02-01 09:00	Stable
Exp-3	2022-12-01 10:00	Deprecated
Historical Experiments to Archive
Exp-4
Exp-5
```

**Code:**

```python
# Example Python script to manage experiment cleanup based on retention policies

import boto3
from datetime import datetime, timedelta

def get_s3_client():
    return boto3.client('s3')

def list_bucket_objects(bucket_name):
    client = get_s3_client()
    response = client.list_objects_v2(Bucket=bucket_name)
    return [obj['Key'] for obj in response.get('Contents', [])]

def delete_old_experiments(bucket_name, days_threshold=90):
    objects = list_bucket_objects(bucket_name)
    current_time = datetime.now()

    # Filter out objects older than the threshold
    old_objects = [obj for obj in objects if (current_time - get_object_creation_date(obj)).days > days_threshold]

    client = get_s3_client()
    for obj_key in old_objects:
        response = client.delete_object(Bucket=bucket_name, Key=obj_key)
        print(f"Deleted object: {obj_key}")

def get_object_creation_date(object_key):
    # This is a placeholder function. In practice, you would use S3 API to fetch the creation date.
    return datetime(2023, 1, 1)  # Mocked for demonstration

delete_old_experiments('ml-experiment-data', days_threshold=90)
```

**Example input / output:**

```text
Input:
- bucket_name: ml-experiment-data
- days_threshold: 90

Output:
- Deleted object: experiment-91011/2022-12-01-10:00/model.h5 (older than 90 days)
```

**Data:**

```json
{
  "bucket_name": "ml-experiment-data",
  "days_threshold": 90,
  "objects": [
    "experiment-1234/2023-01-01-08:00/model.h5",
    "experiment-5678/2023-02-01-09:00/model.h5",
    "experiment-91011/2022-12-01-10:00/model.h5"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initial Experiment Phase
4. Active Experiments
5. Are Experiments Stabilized?
6. Stable Experiments
7. Archive Old Experiments
8. Monitor Performance & Retainfor Audit
9. Archive Historical Data

