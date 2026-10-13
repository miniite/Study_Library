---
type: "practice"
difficulty: "Expert"
---

# How do you design schema evolution across all three layers so a breaking change in a source system doesn't cascade into hundreds of downstream Gold consumers overnight?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Controlling Gold Sprawl](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Controlling_Gold_Sprawl.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of managing schema evolution across the Bronze, Silver, and Gold layers without causing cascading issues, we need to adopt a robust schema migration strategy that minimizes disruptions. This involves understanding the nature of breaking changes in source systems, designing effective migrations, and implementing mechanisms for testing and rollback. Here’s how you can approach this problem:

Understanding Breaking Changes: A breaking change is any alteration to the data model or structure that could affect downstream consumers. Common examples include adding a new required column, changing the type of an existing column, or removing a column.

Designing Migrations: The key is to break down large changes into smaller, manageable steps. Use version control for schemas and implement incremental migration strategies.

Testing and Validation: Ensure that each migration step can be tested in isolation before applying it across all layers. This includes unit testing, integration testing, and end-to-end validation with mock consumers.

Rollback Mechanisms: Have a rollback plan in place to quickly revert any changes if issues arise during the migration process.

**Comparison:**

| Layer | Changes Applied                           |
|-------|-------------------------------------------|
| Bronze | Added `shipping_address` column          |
| Silver | Updated join logic for `sales` table     |
| Gold   | Modified queries to handle new schema    |

**Code:**

```python
# Example of schema version control using a simple incrementing integer
class SchemaVersion:
    def __init__(self, current_version):
        self.current_version = current_version

    def next(self):
        return self.current_version + 1

migration_versions = [SchemaVersion(0), SchemaVersion(1)]
```

**Example input / output:**

```text
# Input: A breaking change to the `Salesforce` source system adding a new required column `shipping_address`
# Output: Incremental migration steps applied in Silver and Gold layers without impacting consumers:
1. Add new column `shipping_address` to Bronze layer.
2. Update Silver transformations that join with `sales` table.
3. Validate changes in a controlled environment before applying to production.
4. Ensure all Gold consumer queries are updated or rewritten to handle the new schema.

# Mock Rollback Plan
- If issues arise, revert to previous version and re-run validation steps.
```

**Data:**

```json
{
  "bronze": {
    "source_systems": ["Salesforce", "CRM"],
    "data_types": ["orders", "customers"]
  },
  "silver": {
    "transformations": [
      {"type": "join", "tables": ["sales", "products"]},
      {"type": "filter", "columns": ["order_date"]}
    ]
  },
  "gold": {
    "consumers": ["BI_analysts", "data_science_team"],
    "metrics": ["total_revenue", "customer_churn_rate"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify Breaking Change
6. Is it a Major Change?
7. Migrate Incrementally
8. Apply to Bronze, Silver, GoldLayers
9. Continue with NormalMigrations
10. Perform Unit & IntegrationTests
11. Deploy Changes to Production
12. Monitor for Issues
13. Issues?
14. Rollback to Previous Version
15. Migrate IncrementallyBackwards
16. Continue with Next Migration

