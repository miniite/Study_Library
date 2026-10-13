---
type: "practice"
difficulty: "Expert"
---

# Your LLM feature's cost is dominated by prompt tokens because every request stuffs the full context. As architect, design the cost-governance strategy across prompt caching, context compression, model routing, and per-tenant budgets.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shadow Mode for Cost Policies](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_Mode_for_Cost_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this role as a Fullstack Architect/Principal, we are tasked with designing a comprehensive cost-governance strategy to manage the high costs associated with prompt tokens in our LLM feature. The goal is to optimize both resource utilization and cost while maintaining performance. We will focus on four key areas: prompt caching, context compression, model routing, and per-tenant budgets.

**Comparison:**

```
Context Size	Model Selected
3000	default
4000	default
6000	high_volume
```

**Code:**

```
Example pseudo-code for per-tenant budget enforcement

class BudgetEnforcer: def init(self, tenant_id, max_cost): self.tenant_id = tenant_id self.max_cost = max_cost

CODE
Copy
def charge(self, cost):
    if self.current_cost + cost > self.max_cost:
        raise ValueError("Exceeded budget limit")
    self.current_cost += cost
Example model routing strategy

class ModelRouter: def init(self): self.models = { "default": "model1", "high_volume": "model2", # more models can be added here based on context and tenant needs }

CODE
Copy
def select_model(self, tenant_id, context_size):
    if tenant_id in self.routing_rules:
        return self.routing_rules[tenant_id]
    elif context_size > 5000:  # Example threshold
        return "model2"
    else:
        return "default"
Example prompt caching mechanism

class PromptCache: def init(self): self.cache = {}

CODE
Copy
def get(self, key):
    if key in self.cache:
        return self.cache[key]
    return None

def set(self, key, value):
    self.cache[key] = value
```

**Example input / output:**

```
Example input to the budget enforcement mechanism

budget_enforcer = BudgetEnforcer("tenant1", 50) for cost in [20, 25, -5]: # Adding and then reducing cost try: budget_enforcer.charge(cost) except ValueError as e: print(e) # Expected to raise when exceeding limit

Example input for model routing strategy

router = ModelRouter() for size in context_sizes: model = router.select_model("tenant1", size) print(f"Selected model for context size {size}: {model}")
```

**Data:**

```
Sample tenant data for budget enforcement

tenants = { "tenant1": {"max_cost": 50}, "tenant2": {"max_cost": 100} }

Example context sizes and corresponding model costs

context_sizes = [3000, 4000, 6000] model_costs = [0.05, 0.1, 0.15]
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[User Request] --> B{Context Size > 5000?};
4. B --> |Yes| C[Select "high_volume" model];
5. B --> |No| D[Select "default" model];
6. D --> E[Prompt Cache Lookup];
7. E --> |Hit| F[Return cached response];
8. E --> |Miss| G[Run LLM with selected model, Charge Tenant];
9. G --> H[Budget Enforcer]

