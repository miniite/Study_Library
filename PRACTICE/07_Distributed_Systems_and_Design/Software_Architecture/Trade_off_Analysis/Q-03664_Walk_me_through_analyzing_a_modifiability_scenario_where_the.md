---
type: "practice"
difficulty: "Deep"
---

# Walk me through analyzing a modifiability scenario where the business wants to spin up a white-labeled instance for a new market in under a month. What architectural sensitivity points (multi-tenancy model, config vs code) does that scenario probe?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scenario Prioritization](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Scenario_Prioritization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When analyzing the modifiability requirements for spinning up a white-labeled instance for a new market in under a month, we need to consider various architectural sensitivity points related to multi-tenancy and configuration versus code. This scenario demands that our architecture supports quick deployment while maintaining isolation between tenants. Let's break down these considerations from first principles.

Firstly, multi-tenant models can significantly impact modifiability. We'll examine whether a shared or dedicated environment is more suitable for this requirement. Secondly, the distinction between configuration and code will affect how we handle customizations without impacting other tenants. We'll explore trade-offs in terms of maintainability, scalability, and security.

**Comparison:**

```
Multi-Tenant Model	Shared Environment	Dedicated Environment
Pros	Cost-effective, easier to manage	Better security, isolation of resources
Cons	Risk of tenant interferences	Higher cost, harder to manage
```

**Code:**

```python
# Example of basic multi-tenant setup using Python with Flask

def get_tenant_config(tenant_id):
    return {
        "name": f"Tenant {tenant_id}",
        "settings": {"theme": "default", "features_enabled": [True, False]}
    }

def configure_environment(app, tenant_config):
    app.config['THEME'] = tenant_config["settings"]["theme"]
    for feature in tenant_config["settings"]["features_enabled"]:
        if feature:
            register_feature(app)
```

**Example input / output:**

```text
Input:
- Tenant configuration data

Output:
- Updated application settings based on tenant's specific needs
- Ensured each tenant has its own isolated environment with no interference from others
```

**Data:**

```json
{
  "tenants": [
    { "id": 1, "name": "Tenant A" },
    { "id": 2, "name": "Tenant B" }
  ]
}
```

**Flow:**

1. Shared
2. Dedicated
3. Multi-Tenant Model
4. Shared or Dedicated?
5. Cost-effective but riskier
6. Better security and isolation
7. Maintain tenant boundaries,monitor carefully
8. Higher initial cost, easier tomanage

