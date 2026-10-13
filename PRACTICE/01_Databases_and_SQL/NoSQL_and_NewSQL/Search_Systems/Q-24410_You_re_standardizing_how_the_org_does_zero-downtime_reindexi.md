---
type: "practice"
difficulty: "Expert"
---

# You're standardizing how the org does zero-downtime reindexing as a platform capability, since every team hand-rolls it dangerously. Design the reindex-as-a-service: alias orchestration, dual-write windows, validation gates, and automatic rollback. What are the correctness invariants you enforce?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Relevance](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Relevance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Zero-downtime reindexing is crucial to maintain high availability and ensure that search operations continue uninterrupted during index updates. As a backend architect, we need to standardize this process across the organization by designing an "reindex-as-a-service" (ReIndexaaS) platform that handles alias orchestration, dual-write windows, validation gates, and automatic rollbacks. The correctness invariants we must enforce include ensuring data consistency, avoiding partial updates, maintaining search availability, and providing robust error handling.

**Comparison:**

| Invariant        | Description                     |
|------------------|---------------------------------|
| Data Consistency | Ensure that data in both indices are consistent during the dual-write process.                  |
| No Partial Writes| Prevent situations where only part of the index is updated, leading to inconsistent data.     |
| Search Availability | Maintain uninterrupted search availability throughout the reindexing process.               |
| Robust Error Handling | Provide clear error messages and fallback mechanisms for handling failures gracefully.      |

**Code:**

```python
# Example of a basic ReIndexaaS API
class ReIndexaaS:
    def __init__(self):
        self.main_index = "main_index"
        self.temp_index = "temp_index"
        self.aliases = {"search_alias": self.main_index}

    def reindex(self, index_name: str) -> bool:
        # Perform a dual-write and alias orchestration
        success, temp_index_exists = self.perform_dual_write(index_name)
        if not success or not temp_index_exists:
            return False

        # Validate the new index
        validation_passed = self.validate_new_index()
        if not validation_passed:
            self.rollback(index_name)
            return False

        # Switch aliases to the new index
        self.switch_aliases(index_name)

        return True

    def perform_dual_write(self, index_name: str) -> (bool, bool):
        # Code for dual-write logic
        pass

    def validate_new_index(self) -> bool:
        # Code for validation logic
        pass

    def rollback(self, index_name: str):
        # Code for rolling back to the old state
        pass

    def switch_aliases(self, index_name: str):
        # Code for switching aliases
        pass
```

**Example input / output:**

```text
| Input             | Output                        |
|-------------------|-------------------------------|
| ReIndexaaS().reindex("movies") | True                          |

Assuming the dual-write and validation steps succeed, the reindex operation will return `True`. If any step fails, it will return `False`.
```

**Data:**

```json
{
  "main_index": "movies",
  "temp_index": "temp_movies",
  "aliases": {
    "search_alias": "movies"
  }
}
```

**Flow:**

1. Success
2. Passed
3. Failed
4. Start ReIndex
5. Perform Dual-Write
6. Create Temp Index
7. Validate New Index
8. Switch Aliases
9. Return True
10. Rollback
11. Return False

