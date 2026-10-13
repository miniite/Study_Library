---
type: "practice"
difficulty: "Deep"
---

# How do you version Gold data-product releases so downstream ML models and dashboards can pin to a stable contract while you evolve the underlying logic?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Controlling Gold Sprawl](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Controlling_Gold_Sprawl.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Medallion architecture, managing the evolution of your data products—especially the "Gold" layer which serves final analysis and ML models—is critical. The challenge is to ensure that downstream systems like ML models and dashboards can rely on a stable contract while evolving underlying logic over time. To achieve this, we version our Gold data releases effectively.

The key idea is to maintain a historical record of both the data and its transformation logic, allowing us to roll back or reference specific versions as needed without breaking the current systems that depend on the latest stable state.

#### Triage
What is being asked: The question revolves around managing versioning in the Gold layer of Medallion architecture to enable downstream systems to pin to a stable contract while evolving the underlying logic.
What the interviewer is probing:
Understanding of how version control can be applied to data products, particularly in a multi-versioned environment.
Knowledge on how to decouple the stable contract from ongoing evolution.
Best practices for handling versioning and ensuring backward compatibility.

#### Assumptions
We are working with a mature data pipeline that includes Gold-level data.
The goal is not just to add new features but also maintain stability and consistency in the data.

#### First-Principles Solution

Versioning Strategy:

Each release of the Gold layer should be tagged with a unique version identifier, such as v1, v2, etc.
A change management process must be established to ensure that any changes are thoroughly tested before deployment.

Data Version Storage:

Store historical versions of the data in a segregated area within the Gold layer or in an adjacent versioned storage area.
Use naming conventions like gold.v1 for version 1, ensuring that each version has a clear and consistent name.

Contract Management:

Define contracts for data schema, transformations, and metadata that are published alongside the data versions.
Ensure that all downstream systems (e.g., ML models, dashboards) can query or reference these contracts using version tags.

Rollback Mechanism:

Implement a rollback strategy to quickly revert to previous versions if issues arise in the latest release.
Use versioning tools and scripts to facilitate this process efficiently.

Testing and Validation:

Set up automated testing frameworks to validate new data transformations against known good states (versions).
Perform rigorous validation before promoting changes to the production environment.

#### Closing Summary

To effectively manage Gold data releases in Medallion architecture, we employ a versioning strategy that ensures stability for downstream systems while allowing for ongoing evolution of underlying logic. By tagging each release with unique identifiers and maintaining clear contracts, we can ensure that critical business operations remain unaffected by changes. Additionally, robust rollback mechanisms provide flexibility to address any issues that may arise post-deployment.

#### Common Mistakes/Follow-Ups
Failing to Document Contracts: Not properly documenting the schema or transformations for each version can lead to downstream systems breaking.
Insufficient Testing: Skipping thorough testing before promoting changes can result in production issues that are hard to debug.
Inadequate Rollback Planning: Lack of a clear rollback strategy means downtime and potential data inconsistencies if changes need to be reverted.

**Comparison:**

```
Version	Timestamp	Data Version	Status
v1	2023-09-15	raw_data_2023	passed
v2	2023-10-01	filtered_data_2023	failed
Schema Version	Fields	Status
2	id, name, value	updated
Transformation Steps	Step	Version	Status
v1	filter_outliers	passed
v2	normalize_values	failed
```

**Code:**

```python
# Example code for versioning in Python
def version_data(data, version):
    return f"v{version}_{data}"

data_v1 = "raw_data_2023"
data_v2 = version_data(data_v1, 1)

print(data_v1)  # Output: raw_data_2023
print(data_v2)  # Output: v1_raw_data_2023
```

**Example input / output:**

```text
Input:
- New data version to be released: v3
- Transformation steps for v3

Output:
- Updated contracts:
  - Schema version: 2
  - Transformation steps status: all passed or updated to reflect reality

Dry-run Request:
- Validate new transformation step in isolated environment before promotion.
```

**Data:**

```json
{
    "gold": {
        "versioned_data": [
            {"timestamp": "2023-09-15", "data_version": "v1"},
            {"timestamp": "2023-10-01", "data_version": "v2"}
        ]
    },
    "contracts": {
        "schema": {"fields": ["id", "name", "value"], "version": 2},
        "transformations": [
            {"step": "filter_outliers", "version": 1, "status": "passed"},
            {"step": "normalize_values", "version": 1, "status": "failed"}
        ]
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Receive New Data
6. Is Transformation Needed?
7. Apply Transformations
8. Generate Versioned Data
9. Update Contracts andMetadata
10. Publish to Gold Layer
11. Monitor for Issues
12. Issues Detected?
13. Rollback to Previous Version
14. Continue Monitoring
15. Automated Testing

