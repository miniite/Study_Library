---
type: "practice"
difficulty: "Expert"
---

# Probe: your metadata event schema needs to evolve as you add features. How do you version the metadata model itself and migrate millions of existing entities without downtime or breaking consumers of the metadata API?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Catalog Consistency Model](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Catalog_Consistency_Model.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle evolving a metadata event schema while maintaining uptime and ensuring data integrity, we need to adopt a structured approach involving versioning and a phased migration strategy. The core idea is to maintain backward compatibility while incrementally introducing new features.

First, let's understand the key components:

Versioning: We'll introduce multiple versions of our metadata model.
Phased Migration: Gradually migrate existing entities from older to newer schema versions without disrupting ongoing operations.

#### Cons
umer Protection: Ensure that consumers using the metadata API are shielded from changes during migration.

Here’s a step-by-step breakdown:

Identify Schema Changes and Versioning Strategy:

Determine which fields or logic need updates, and classify them into different version numbers (e.g., v1, v2).
Decide on a mechanism for mapping older data to new schema formats if necessary.

Design Migration Scripts:

Develop scripts that can convert data from the old format to the new format.
These scripts should handle any necessary transformations and ensure that no data is lost or corrupted during migration.

Create a Staging Environment:

Set up a staging environment where you can test migrations without affecting production systems.
This includes setting up separate databases or tables for old and new schema versions.

Implement a Migration Strategy:

Start with a small, non-critical set of entities to validate the migration scripts.
Gradually expand the scope until all entities are migrated.

Update Consumers Gradually:

Once migrations are complete, update consumer applications one by one to support new schema versions.
Monitor for any issues and rollback if necessary.

Maintain a Rollback Plan:

Have clear steps for reverting changes in case something goes wrong during migration.

Document Changes:

Maintain detailed documentation of the changes made, including scripts used, configurations, and any special considerations.
Detailed Steps

Schema Definition:

SQL
CREATE SCHEMA IF NOT EXISTS `metadata_v2` ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

Script for Data Migration:

PYTHON
def migrate_entity(entity):
    old_data = get_old_data(entity)
    new_data = transform_data(old_data)
    save_new_data(new_data)

def transform_data(data):
    # Implement logic to convert data from v1 to v2 schema
    pass

entities = get_all_entities()
for entity in entities:
    migrate_entity(entity)

Staging Environment Setup:

SQL
-- In the staging environment
CREATE DATABASE metadata_staging;
USE metadata_staging;

-- Copy existing data into a new table with v2 schema
SELECT * INTO TABLE `metadata_v2.entities` FROM `current.entities`;

Migration Execution:

Run migration scripts in a controlled manner.
Monitor for any issues and address them promptly.

#### Cons
umer Update Strategy:

PYTHON
def update_consumer_api():
    # Logic to update consumer API to support new schema version
    pass

#### cons
umers = get_all_consumers()
for consumer in consumers:
    if should_migrate(consumer):
        update_consumer_api(consumer)

#### Common Mistakes
and Follow-Ups
Ignoring Backward Compatibility: Always ensure that the migration scripts are backward compatible to avoid breaking existing integrations.
Inadequate Testing: Thoroughly test migrations in a staging environment before applying them to production.
Failing to Document Changes: Maintain detailed records of all changes for future reference and troubleshooting.

By following these steps, we can safely evolve our metadata model without downtime or disrupting consumers.

**Comparison:**

| Entity | Old Schema          | New Schema             |
|--------|---------------------|-----------------------|
| entity_id | Integer             | Integer               |
| name     | String              | String                |
| version  | String              | String                |
|         |                     | description           |

**Example input / output:**

```

Initial State:

JSON
Copy
{
  "entity_id": 1,
  "name": "OldEntity",
  "version": "v1"
}

After Migration:

JSON
Copy
{
  "entity_id": 1,
  "name": "MigratedEntity",
  "description": "Updated with new fields and logic.",
  "version": "v2"
}
```

**Data:**

```

CREATE SCHEMA IF NOT EXISTS metadata_v2 ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CODE
Copy

```sql
-- In the staging environment
CREATE DATABASE metadata_staging;

USE metadata_staging;

SELECT * INTO TABLE `metadata_v2.entities` FROM `current.entities`;
```

**Flow:**

1. Identify Schema Changes
2. Define Versioning Strategy
3. Design Migration Scripts
4. Staging Environment Setup
5. Migrate Entities Gradually
6. Update Consumers Gradually
7. Document Changes

