---
type: "practice"
difficulty: "Deep"
---

# How would you design a governance layer so that every feature produced by any Spark pipeline in the org has enforceable ownership lineage freshness SLA and a data contract before it can feed a production model?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Pipeline Platforms](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a governance layer ensuring every feature produced by any Spark pipeline in the organization adheres to enforced ownership, lineage, freshness Service Level Agreements (SLA), and data contracts before feeding into production models, we need to break down the problem into several key components. First, let's summarize the core concepts and then proceed with the detailed solution.

#### Triaging the Question

The question is asking for a governance framework that enforces compliance across all Spark pipelines in an organization. The primary concerns are:

Ownership: Ensuring features have clear ownership.
Lineage: Tracking how each feature is derived from raw data through various transformations.
Freshness SLA: Guaranteeing that the data meets freshness requirements before being used.
Data Contract: Verifying that the data conforms to predefined schemas and quality standards.

#### Assumptions
The organization already has a robust data pipeline infrastructure using tools like Apache Spark.
There is a centralized Airflow instance managing pipelines.
Data contracts are defined in JSON or similar formats.
Step-by-Step Solution

Ownership Layer:

Each feature should have a designated owner who is responsible for the quality and accuracy of the data.
Ownership can be enforced by tagging each pipeline with an owner, and this information should be stored in a metadata repository (e.g., Airflow metadata DB).

Lineage Tracking:

Lineage must be meticulously tracked to ensure that any change in data can trace back to its origin.
Implement a lineage tracking mechanism using tools like Apache Atlas or custom solutions within the existing pipeline infrastructure.

Freshness SLA Enforcement:

Define freshness SLAs for each feature and enforce them at runtime.
Use Airflow's sla_miss functionality to alert stakeholders when data does not meet the SLA.

Data Contract Validation:

Validate incoming data against predefined schemas or contracts before it is used in any pipeline.
Integrate schema validation tools like Apache Beam’s Schema Enforcer into the pipeline workflow.

#### Implementation

Ownership Tagging:

PYTHON
# Example metadata tagging for Airflow tasks
from airflow.models import BaseOperator, XCom

class OwnedFeatureTask(BaseOperator):
    def __init__(self, owner=None, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.owner = owner

    def execute(self, context):
        # Tagging the task with an owner
        context['task_instance'].xcom_push(key='owner', value=self.owner)
        return "Feature generated"

Lineage Tracking:

Utilize Apache Atlas or custom lineage tracking.
PYTHON
from atlas import Client

client = Client()

def track_lineage(feature_name, input_data):
    client.track(feature_name, source=input_data)

Freshness SLA Enforcement:

PYTHON
from datetime import datetime

def check_freshness_sla(feature_name, required_age):
    current_time = datetime.now()
    last_updated = get_last_update_time(feature_name)  # Assume a function to fetch the last update time
    if (current_time - last_updated).days > required_age:
        return False
    return True

def ensure_freshness():
    for feature in list_of_features:  # Assume this is a predefined list of features
        if not check_freshness_sla(feature, SLA_days):
            raise Exception(f"Feature {feature} did not meet the freshness SLA.")

Data Contract Validation:

PYTHON
from schema_validator import validate

def validate_data_contract(data, contract):
    return validate(data, contract)

# Example usage
data = {...}
contract = {...}
if not validate_data_contract(data, contract):
    raise Exception("Validation failed")

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By integrating these components, we can ensure that every feature produced by Spark pipelines has enforceable governance controls. This includes ownership tagging, lineage tracking, freshness SLA enforcement, and data contract validation.

#### Common Mistakes
Overlooking the need for detailed documentation of each step in the pipeline.
Not implementing real-time monitoring and alerting mechanisms.
Ignoring edge cases such as handling stale features or dynamically changing contracts.

#### Follow-Ups
How would you handle scenarios where multiple owners are assigned to a feature?
What measures would you take if a SLA is frequently missed?
How do you ensure that the lineage tracking does not significantly impact performance?

**Comparison:**

```
Yes
No
Yes
No
Yes
No
Ownership Tagging
Is Owner Known?
Tag Task with Owner
Alert for Missing Owner
Lineage Tracking
Is Lineage Tracked?
Ensure Data Freshness SLA
Alert for Untracked Lineage
Validate Data Contract
Validation Passed?
Data Ready for Use
Alert for Failed Validation
```

**Code:**

```

from airflow.models import BaseOperator, XCom

class OwnedFeatureTask(BaseOperator): def init(self, owner=None, *args, **kwargs): super().init(*args, **kwargs) self.owner = owner

CODE
Copy
def execute(self, context):
    context['task_instance'].xcom_push(key='owner', value=self.owner)
    return "Feature generated"
CODE
Copy

```python
from datetime import datetime

def check_freshness_sla(feature_name, required_age):
    current_time = datetime.now()
    last_updated = get_last_update_time(feature_name)  # Assume a function to fetch the last update time
    if (current_time - last_updated).days > required_age:
        return False
    return True

def ensure_freshness():
    for feature in list_of_features:  # Assume this is a predefined list of features
        if not check_freshness_sla(feature, SLA_days):
            raise Exception(f"Feature {feature} did not meet the freshness SLA.")
PYTHON
Copy
from schema_validator import validate

def validate_data_contract(data, contract):
    return validate(data, contract)

# Example usage
data = {...}
contract = {...}
if not validate_data_contract(data, contract):
    raise Exception("Validation failed")
```

**Example input / output:**

| Feature | Last Updated (days ago) | SLA Met? |
|---------|------------------------|----------|
| feature1 | 15                     | Yes      |
| feature2 | 45                     | No       |

**Data:**

```json
{
  "list_of_features": ["feature1", "feature2"],
  "SLA_days": 30
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start
8. Ownership Tagging
9. Is Owner Known?
10. Tag Task with Owner
11. Alert for Missing Owner
12. Lineage Tracking
13. Is Lineage Tracked?
14. Ensure Data Freshness SLA
15. Alert for Untracked Lineage
16. Validate Data Contract
17. Validation Passed?
18. Data Ready for Use
19. Alert for Failed Validation
20. End
21. End

