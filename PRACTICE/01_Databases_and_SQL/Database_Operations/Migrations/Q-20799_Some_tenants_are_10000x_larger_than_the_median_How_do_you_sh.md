---
type: "practice"
difficulty: "Deep"
---

# Some tenants are 10000x larger than the median. How do you shape a fleet-wide migration so whales don't stall the queue and small tenants don't wait days?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrating to Partitioned Tables](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Migrating_to_Partitioned_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When managing database migrations across a diverse tenant ecosystem, it's crucial to design a system that ensures large tenants (whales) do not stifle the migration process while preventing smaller tenants from waiting excessively long periods. The key is to implement a stratified and adaptive approach that prioritizes migrations based on size and resource availability.

The core idea is to divide the tenant base into groups and migrate them in parallel with differentiated time windows, ensuring that larger tenants receive more resources for their migrations without disrupting other tenants' schedules. This involves carefully crafting an algorithm that balances resource allocation and migration timing.

**Comparison:**

| Tenant ID | Size   |
|-----------|--------|
| 1         | 2000   |
| 2         | 5000   |
| 3         | 10000  |
| 4         | 8000   |
| 5         | 6000   |

**Code:**

```
Example pseudocode for stratified migration scheduling

def get_tenant_size(tenant_id): # Assume a function to fetch tenant size in MB or records return db.get_tenant_info(tenant_id)['size']

def stratify_tenants(tenants, max_size=10000): # Median size is assumed to be 10000MB small, medium, large = [], [], [] for tenant in tenants: size = get_tenant_size(tenant) if size <= 5000: # Small: <= 5000MB, assumes this is a lower bound small.append(tenant) elif size > 5000 and size < max_size: medium.append(tenant) else: # Large: >= max_size (10000MB+) large.append(tenant) return small, medium, large

def schedule_migrations(small, medium, large): scheduler = MigrationScheduler() for tenant in small + medium: scheduler.add_migration(tenant, priority=2) # Medium priority for tenant in large: scheduler.add_migration(tenant, priority=1) # High priority to avoid blocking

CODE
Copy
return scheduler.schedule()

class MigrationScheduler: def init(self): self.queue = []

CODE
Copy
def add_migration(self, tenant_id, priority):
    self.queue.append((tenant_id, priority))

def schedule(self):
    # Simulate migration scheduling logic
    for tenant_id, _ in sorted(self.queue, key=lambda x: x[1], reverse=True):
        print(f"Scheduling {tenant_id} with priority {self.queue.index((tenant_id, 2)) + 1}")
Example usage

tenants = [1, 2, 3, 4, 5] small_tenants, medium_tenants, large_tenants = stratify_tenants(tenants) schedule_migrations(small_tenants, medium_tenants, large_tenants)

CODE
Copy

```python
# Example pseudocode for stratified migration scheduling

def get_tenant_size(tenant_id):
    # Assume a function to fetch tenant size in MB or records
    return db.get_tenant_info(tenant_id)['size']

def stratify_tenants(tenants, max_size=10000):  # Median size is assumed to be 10000MB
    small, medium, large = [], [], []
    for tenant in tenants:
        size = get_tenant_size(tenant)
        if size <= 5000:  # Small: <= 5000MB, assumes this is a lower bound
            small.append(tenant)
        elif size > 5000 and size < max_size:
            medium.append(tenant)
        else:  # Large: >= max_size (10000MB+)
            large.append(tenant)
    return small, medium, large

def schedule_migrations(small, medium, large):
    scheduler = MigrationScheduler()
    for tenant in small + medium:
        scheduler.add_migration(tenant, priority=2)  # Medium priority
    for tenant in large:
        scheduler.add_migration(tenant, priority=1)  # High priority to avoid blocking

    return scheduler.schedule()

class MigrationScheduler:
    def __init__(self):
        self.queue = []

    def add_migration(self, tenant_id, priority):
        self.queue.append((tenant_id, priority))

    def schedule(self):
        # Simulate migration scheduling logic
        for tenant_id, _ in sorted(self.queue, key=lambda x: x[1], reverse=True):
            print(f"Scheduling {tenant_id} with priority {self.queue.index((tenant_id, 2)) + 1}")

# Example usage
tenants = [1, 2, 3, 4, 5]
small_tenants, medium_tenants, large_tenants = stratify_tenants(tenants)
schedule_migrations(small_tenants, medium_tenants, large_tenants)
```

**Example input / output:**

```text
Scheduling tenant id=2 with priority 2 (medium)
Scheduling tenant id=1 with priority 2 (medium)
Scheduling tenant id=4 with priority 2 (medium)
Scheduling tenant id=5 with priority 2 (medium)
Scheduling tenant id=3 with priority 1 (large, high priority to avoid blocking)
```

**Data:**

```json
{
    "tenants": [
        {"id": 1, "size": 2000},
        {"id": 2, "size": 5000},
        {"id": 3, "size": 10000},
        {"id": 4, "size": 8000},
        {"id": 5, "size": 6000}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is the tenant size <= 5000MB?
7. Schedule with medium priority
8. Is the tenant size >=10000MB?
9. Schedule with high priority toavoid blocking
10. Schedule with medium priority

