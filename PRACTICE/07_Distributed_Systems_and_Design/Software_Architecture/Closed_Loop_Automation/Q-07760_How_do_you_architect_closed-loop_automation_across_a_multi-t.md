---
type: "practice"
difficulty: "Expert"
---

# How do you architect closed-loop automation across a multi-tenant SaaS platform so a remediation for one tenant can't degrade another?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Limits of Automation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Limits_of_Automation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AIOps architecture, ensuring that automated remediations are tenant-specific and isolated from each other is critical to maintaining service quality and avoiding cascading failures. Closed-loop automation involves not only automating responses but also continuously monitoring and refining those responses based on outcomes. To architect this across a multi-tenant SaaS platform without one tenant's issues affecting another, we need to consider several layers: isolation at the data level, strict control over workflows, robust monitoring, and efficient feedback loops.

The core idea is to design a system where each tenant’s environment and associated automation processes are segmented and isolated from others. This ensures that any remediation actions initiated for one tenant do not impact another, while still allowing for shared resources or common services if needed.

**Code:**

```python
# Example pseudo-code for isolation mechanism in Python

class Tenant:
    def __init__(self, tenant_id):
        self.tenant_id = tenant_id
        self.data_store = DataStore(tenant_id)
        self.workflow_engine = WorkflowEngine(tenant_id)

def execute_remediation(tenant: Tenant, issue_type):
    if not tenant.isolation_check(issue_type):
        return False  # If isolation fails, remediation is not executed

    tenant.workflow_engine.execute(issue_type)
    return True
```

**Example input / output:**

| Tenant ID | Action Initiated | Remediation Execution |
|-----------|------------------|-----------------------|
| tenant1   | Memory Leak      | Success               |
| tenant2   | CPU Overload     | Failed                |
| tenant3   | Disk Full        | Success               |

**Data:**

```yaml
# Example YAML for data store configuration per tenant
tenant1:
  data_store_config:
    - key: "instance_id"
      value: "1234567890abcdef1234567890abcded"

tenant2:
  data_store_config:
    - key: "instance_id"
      value: "0987654321fedcba0987654321fecedc"
```

