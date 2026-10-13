---
type: "practice"
difficulty: "Deep"
---

# Your model's input table is defined by a dbt model owned by analytics engineering. How do you coordinate so their refactor doesn't silently break your features?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Central vs Domain Feature Modeling](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Central_vs_Domain_Feature_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When integrating with an external dbt model managed by Analytics Engineering, ensuring that our machine learning (ML) pipeline remains stable and robust is paramount. The key challenge lies in coordinating changes to the input schema or data without causing unexpected disruptions in our models. To address this, we need a comprehensive approach that includes clear communication, versioning, and continuous integration (CI).

First, let's lay out the core idea: by establishing a well-defined interface between the Analytics Engineering team and our ML pipeline, we can ensure seamless coordination and maintain data integrity.

**Comparison:**

```
dbt Model	Versioned Schema
user_profile_v1	id, name, age
user_profile_v2	id, name, age, email
```

**Code:**

```python
# Example of a dbt model schema
from dbt import Schema

user_profile = Schema(
    id='id',
    name='name',
    age='age'
)
```

**Flow:**

1. Analytics Engineering Team
2. Refactor dbt Model
3. Versions Management
4. Maintain Data Integrity
5. Notify ML Engineers
6. Continuous Integration Checks
7. Update ML Pipeline

