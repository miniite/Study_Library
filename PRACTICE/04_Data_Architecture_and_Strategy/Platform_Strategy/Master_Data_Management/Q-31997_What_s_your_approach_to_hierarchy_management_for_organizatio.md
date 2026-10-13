---
type: "practice"
difficulty: "Expert"
---

# What's your approach to hierarchy management for organizations (legal entity vs billing vs household vs corporate family tree), and how do you version hierarchies that change through M&A?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-domain MDM Programs](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Multi-domain_MDM_Programs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a data-driven organization, managing hierarchies is critical for ensuring consistency, accuracy, and efficient data governance. When dealing with an organization's legal entity hierarchy, billing structure, household composition, and corporate family tree, we need to ensure that these hierarchies are well-managed and versioned effectively, especially in the context of mergers and acquisitions (M&A). Here’s my approach:

#### Overview
The key idea is to maintain a clear, scalable, and version-controlled hierarchy structure that supports various organizational perspectives like legal entities, billing units, households, and corporate family trees. This involves creating a unified data model that captures the necessary hierarchies and their relationships.

#### Triage
The question is probing how we manage and version complex hierarchical structures in an organization. It requires understanding the different types of hierarchies involved (legal entity vs billing vs household) and how these change through M&A activities. Key assumptions include:

Hierarchies are dynamic and can evolve over time.
Data quality and consistency are critical.
The solution must support historical data for audit purposes.

#### First-Principles Solution

Unified Hierarchy Model: Create a unified hierarchy model that supports different perspectives (legal entities, billing units, households) while ensuring common attributes like entity IDs, parent-child relationships, and timestamps.
Versioning Strategy: Implement version control to track changes over time. Use a hierarchical database or a relational schema with temporal tables to capture historical data.
Change Management Workflow: Establish a robust change management process that involves tracking M&A activities, updating the hierarchy model, and ensuring consistency across all perspectives.

Detailed Steps:

Define Hierarchical Structure: Start by defining the core entities (legal entities, billing units, households) and their relationships. Use domain expertise to establish the rules governing these hierarchies.
SQL
CREATE TABLE LegalEntities (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    parent_id INT REFERENCES LegalEntities(id)
);
Temporal Tables: For versioning, implement temporal tables that capture historical data.
SQL
CREATE TABLE LegalEntities_History (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    parent_id INT,
    effective_date DATE NOT NULL,
    end_date DATE DEFAULT '9999-12-31'
);
Change Management: Develop a change management process that involves:
Identifying and logging M&A activities.
Updating the hierarchy model with new relationships or changes.
Ensuring consistency across all hierarchies (legal entity, billing unit, household).
Audit Trails: Maintain detailed logs of changes to support auditing and compliance.

#### Closing Summary
A robust approach involves creating a unified hierarchy model that supports multiple organizational perspectives while implementing version control to handle dynamic changes through M&A activities. This ensures data consistency and accuracy across the organization.

#### Common Mistakes/Follow-Ups

Ignoring Historical Data: Failing to maintain historical records can lead to data inconsistencies.
Inadequate Change Management: Lack of a formal process for managing hierarchy updates can result in errors and misalignments.
Overcomplicating the Model: Sometimes, overly complex models can become difficult to manage. Striking a balance between simplicity and comprehensiveness is crucial.

**Comparison:**

| ID | Name       | Parent ID |
|----|------------|-----------|
| 1  | Parent Co  | NULL      |
| 2  | Subsidiary A | 1        |
| 3  | Billing Unit X | 2     |
| 4  | Household Y   | 3        |
| ID | Name       | Parent ID | Effective Date | End Date    |
|----|------------|-----------|----------------|-------------|
| 1  | Parent Co  | NULL      | 2022-01-01     | 9999-12-31  |
| 2  | Subsidiary A | 1        | 2022-01-01     | 9999-12-31  |

**Code:**

```sql
-- Example SQL Schema for LegalEntities
CREATE TABLE LegalEntities (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    parent_id INT REFERENCES LegalEntities(id)
);

-- Temporal Table for Historical Data
CREATE TABLE LegalEntities_History (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    parent_id INT,
    effective_date DATE NOT NULL,
    end_date DATE DEFAULT '9999-12-31'
);
```

**Example input / output:**

```text
Input:
- M&A activity: Parent Co acquires a new legal entity.
Output:
1. Update LegalEntities table to include the new legal entity.
2. Add historical records in LegalEntities_History if necessary.
3. Adjust billing units and households as required.

Request: Log M&A of Parent Co acquiring Subsidiary B
Response:
- Updated LegalEntities with Subsidiary B details.
- Added historical record for any changes in the hierarchy structure.
```

**Data:**

```json
{
  "legalEntities": [
    {
      "id": 1,
      "name": "Parent Co",
      "parent_id": null
    },
    {
      "id": 2,
      "name": "Subsidiary A",
      "parent_id": 1
    }
  ],
  "billingUnits": [
    {
      "id": 3,
      "name": "Billing Unit X",
      "parent_id": 2
    }
  ],
  "households": [
    {
      "id": 4,
      "name": "Household Y",
      "parent_id": 3
    }
  ]
}
```

**Flow:**

1. Identify M&A Activity
2. Log Activity in ChangeManagement System
3. Update LegalEntities Table
4. Verify Changes Across AllHierarchies
5. Maintain Historical Records(LegalEntities_History)

