---
type: "practice"
difficulty: "Expert"
---

# What is the expand-and-contract (parallel change) pattern for backward-compatible migrations?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In cloud-native architectures, the "expand-and-contract" or "parallel change" migration pattern is a strategic approach to making backward-compatible changes in your application while ensuring minimal downtime and preserving data integrity. This method allows you to incrementally phase in new features or infrastructure without breaking existing functionality. The core idea involves deploying both old and new components of your system side-by-side until the transition is complete.

This pattern is particularly useful for migrations involving major version upgrades, changes in database schemas, or refactoring complex services into microservices. To effectively implement this pattern, you need to carefully plan the migration steps, manage dependencies, and ensure that both the old and new components can coexist without interfering with each other.

**Comparison:**

```
Phase	Action	Old Service	New Service
1	Initial Setup	Running	Not running
2	Gradual Rollout	Running	Running, Low usage
3	Full Utilization	Running	Running, High usage
```

**Code:**

```python
# Example of a service using the expand-and-contract pattern

class OldService:
    def process_request(self, request):
        # Old processing logic
        return "Processed by old service"

class NewService:
    def process_request(self, request):
        # New processing logic
        return "Processed by new service"

def setup_migration(old_service: OldService, new_service: NewService) -> None:
    # Ensure both services are running simultaneously during the migration
    while not is_migration_complete():
        try:
            if should_use_new_service():
                result = new_service.process_request(request)
            else:
                result = old_service.process_request(request)
            handle_result(result)
        except Exception as e:
            log_error(e)

def is_migration_complete() -> bool:
    # Check the current migration status
    return True  # This would be determined by some external logic

def should_use_new_service() -> bool:
    # Determine whether to use new service based on conditions
    return random.random() > 0.5  # For simplicity, this is a random decision

def handle_result(result: str) -> None:
    # Handle the result of processing request
    print(result)

# Example usage
setup_migration(OldService(), NewService())
```

**Example input / output:**

```text
Input:
- Simulate incoming requests to the service.

Output:
- For each request, the output alternates between "Processed by old service" and "Processed by new service" until a majority of the traffic is directed to the new service.
```

**Data:**

```json
{
  "migration_status": {
    "old_service_requests_processed": 1000,
    "new_service_requests_processed": 500,
    "total_requests": 2000
  }
}
```

**Flow:**

1. Initial Service
2. Phase 1 - Setup
3. Phase 2 - Gradual Rollout(50/50)
4. Phase 3 - Full Utilization (100%New)

