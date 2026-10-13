---
type: "practice"
difficulty: "Expert"
---

# Define correlated failure domains for a system spanning three AZs and two regions. How do you identify hidden shared-fate dependencies (DNS, config store, secrets, a single control plane) that turn a partial outage into a total one?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a system across multiple availability zones (AZs) and regions, it's crucial to identify correlated failure domains to ensure resilience against potential outages. These correlated failures can arise from shared-fate dependencies like DNS services, configuration stores, secrets management, or single control planes that might affect the entire system if they fail. The goal is to minimize the risk of a partial outage turning into a total one.

The question requires us to pinpoint these hidden dependencies and understand their impact on system reliability. We need to consider the architecture, dependencies between services, and potential points of failure. By identifying such dependencies from first principles, we can design more robust systems that are less prone to cascading failures.

**Comparison:**

```
Service	Dependencies
web_app	DNS
database
config_store	DNS, secrets_manager
secrets_manager	config_store
control_plane	config_store
```

**Code:**

```python
# Example code snippet to illustrate dependency checking in a Python script (not executable)
def check_dependencies(service):
    if service == "DNS":
        return ["config_store", "secrets_manager"]
    elif service == "config_store":
        return ["DNS", "secrets_manager", "control_plane"]
    # Add more conditions as necessary
    else:
        return []
```

**Example input / output:**

| Service | Dependencies |
|---------|--------------|
| web_app  | DNS          |
| database |              |
| config_store | DNS, secrets_manager   |
| secrets_manager | config_store |
| control_plane | config_store         |

**Data:**

```json
{
  "services": {
    "web_app": {"dependencies": ["DNS"]},
    "database": {"dependencies": []},
    "config_store": {"dependencies": ["DNS", "secrets_manager"]},
    "secrets_manager": {"dependencies": ["config_store"]},
    "control_plane": {"dependencies": ["config_store"]}
  }
}
```

**Flow:**

1. web_app
2. DNS
3. config_store
4. secrets_manager
5. control_plane
6. database
7. None

