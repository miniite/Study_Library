---
type: "practice"
difficulty: "Deep"
---

# Model a class design for enforcing tenant data isolation at the ORM/repository layer so no query can ever forget the tenant filter; how do you make the safe path the default and the unsafe path require an explicit opt-out?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tenant Isolation in Repositories](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this low-level design (LLD) question, we need to ensure that every query made through our ORM/repository layer is tenant-specific by default. This means that any operation that retrieves or modifies data should automatically include a tenant filter unless explicitly opted out of. The goal is to make the safe path—the one where queries respect the tenant boundaries—the default behavior and enforce this safety with as little overhead as possible.

#### Triage

The question requires us to design a class structure in an ORM/repository layer that ensures all operations are filtered by tenant ID by default, with explicit support for opt-out scenarios. This means we need to consider how to implement tenant filters efficiently while ensuring they do not interfere with performance or introduce bugs. We'll assume the use of a database with tenant-specific schemas and that our ORM framework supports dynamic filtering.

#### First-Principles Solution
Tenant Context: Each operation should start with an awareness of the current tenant context.
Default Tenant Filter: By default, any query should include the tenant filter.
Explicit Opt-Out Mechanism: Provide a clear and explicit method to opt out of the tenant filter if needed.

#### Implementation

We will implement this using a combination of static and dynamic methods:

PYTHON
class Repository:
    def __init__(self, tenant_id: str):
        self.tenant_id = tenant_id

    def get(self, query_params: dict) -> list:
        # Default behavior: apply tenant filter by default
        return self._get_with_tenant_filter(query_params)

    def _get_with_tenant_filter(self, query_params: dict) -> list:
        # Apply the tenant filter here (e.g., using a dynamic query builder)
        filtered_query_params = {**query_params, "tenant_id": self.tenant_id}
        return self._execute_query(filtered_query_params)

    def get_without_tenant(self, query_params: dict) -> list:
        # Explicit opt-out mechanism
        return self._get_with_tenant_filter({})

class DynamicQueryBuilder:
    def __init__(self):
        pass

    def build_query(self, params: dict) -> str:
        # Construct the SQL query with appropriate filters based on parameters
        sql = "SELECT * FROM table WHERE tenant_id = :tenant_id"
        return sql.format(tenant_id=params.get("tenant_id"))

class Executor:
    def __init__(self):
        pass

    def execute_query(self, query: str) -> list:
        # Simulate database execution (this would be replaced with actual DB interaction)
        results = []
        for _ in range(10):  # Simplified data
            results.append({"id": _, "tenant_id": "TENANT_1", "data": f"Data_{_}"})
        return results

# Example usage
repo = Repository(tenant_id="TENANT_1")
print(repo.get({"some_param": "value"}))  # Default tenant filter applied

no_tenant_repo = repo.get_without_tenant({"some_param": "value"})
print(no_tenant_repo)  # No tenant filter applied

#### Closing Summary
and Common Mistakes/Follow-Ups
Default Tenant Filter: Ensuring that the default behavior is to apply the tenant filter makes the safe path the default. This reduces the likelihood of bugs due to accidentally omitting a critical piece of information.
Explicit Opt-Out Mechanism: Providing an explicit opt-out mechanism ensures flexibility and allows developers to make informed decisions when they need to bypass the tenant filter, but only after understanding the potential implications.

#### Performance Considerations
While the default behavior is efficient, we should also consider performance during the opt-out scenarios. Overhead from unnecessary filtering can be minimized by optimizing query execution paths.

#### Common Mistakes
Neglecting Default Tenant Filter: Failing to enforce tenant filters by default and relying solely on developers to remember to include them.
Lack of Explicit Opt-Out Mechanism: Not providing a clear way to opt out, leading to potential bugs or security vulnerabilities.
Inadequate Performance Considerations: Overhead from unnecessary filtering can lead to performance degradation.

**Comparison:**

| Tenant ID | Query Params   | Result |
|-----------|----------------|--------|
| TENANT_1  | {"some_param": "value"} | Default Tenant Filter Applied |
|          | {}             | No Tenant Filter Applied      |

**Code:**

```python
class Repository:
    def __init__(self, tenant_id: str):
        self.tenant_id = tenant_id

    def get(self, query_params: dict) -> list:
        return self._get_with_tenant_filter(query_params)

    def _get_with_tenant_filter(self, query_params: dict) -> list:
        filtered_query_params = {**query_params, "tenant_id": self.tenant_id}
        return self._execute_query(filtered_query_params)

    def get_without_tenant(self, query_params: dict) -> list:
        return self._get_with_tenant_filter({})

class DynamicQueryBuilder:
    def __init__(self):
        pass

    def build_query(self, params: dict) -> str:
        sql = "SELECT * FROM table WHERE tenant_id = :tenant_id"
        return sql.format(tenant_id=params.get("tenant_id"))

class Executor:
    def execute_query(self, query: str) -> list:
        results = []
        for _ in range(10):
            results.append({"id": _, "tenant_id": "TENANT_1", "data": f"Data_{_}"})
        return results
```

**Example input / output:**

```text
repo = Repository(tenant_id="TENANT_1")
print(repo.get({"some_param": "value"}))  # Default tenant filter applied

no_tenant_repo = repo.get_without_tenant({"some_param": "value"})
print(no_tenant_repo)  # No tenant filter applied
```

**Data:**

```json
{
  "tenant_id": "TENANT_1",
  "query_params": {
    "some_param": "value"
  }
}
```

**Flow:**

1. Yes
2. No
3. Repository
4. DynamicQueryBuilder
5. Is Opt-Out Needed?
6. Execute Query Without TenantFilter
7. Execute Query With TenantFilter
8. Return Results

