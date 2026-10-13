---
type: "practice"
difficulty: "Deep"
---

# How do you distinguish a data owner from a data steward from a data custodian and how do you assign these roles so accountability is unambiguous at scale?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Canonical Model vs Domain Autonomy](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Canonical_Model_vs_Domain_Autonomy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a comprehensive data governance framework, roles such as data owner, data steward, and data custodian play distinct yet interrelated parts in ensuring data quality, integrity, and compliance. A data owner is responsible for defining the business context, legal ownership, and strategic direction of the data assets. A data steward oversees the lifecycle management of these data assets, focusing on metadata standards, data quality, and access control. A data custodian ensures that the physical storage, accessibility, and day-to-day operations of the data align with governance policies.

To assign roles effectively, it's crucial to understand their distinctions and overlaps:

Data Owner: This role is typically held by a business domain expert or executive who defines how the data will be used in line with broader organizational goals. They are accountable for the strategic value of the data.
Data Steward: The steward acts as the technical and operational leader, ensuring that the data assets meet governance requirements. They manage data quality, lifecycle management (retention policies), and compliance with regulatory frameworks.
Data Custodian: This role focuses on the physical storage and accessibility of data. Custodians ensure that data is accessible to authorized users while maintaining security protocols.

**Comparison:**

```
Role	Responsibilities	Example Actions
Data Owner	Defines business context and strategy.	Define how data will be used to maximize customer lifetime value.
Data Steward	Manages metadata, ensures data quality.	Set up ISO standards for metadata; implement real-time validation rules.
Data Custodian	Ensures physical storage & accessibility.	Implement two-factor authentication; use encrypted cloud storage solutions.
```

**Code:**

```text
# Example Code for Role Assignment
class DataOwner:
    def __init__(self, business_domain):
        self.business_domain = business_domain

    def define_strategy(self):
        print(f"Defining strategy for {self.business_domain} data.")

class DataSteward:
    def __init__(self, data_owner):
        self.data_owner = data_owner

    def manage_metadata(self):
        print("Managing metadata standards and policies.")

    def ensure_quality(self):
        print("Ensuring data quality and integrity.")

class DataCustodian:
    def __init__(self, data_steward):
        self.data_steward = data_steward

    def maintain_accessibility(self):
        print("Maintaining accessibility and security of the data.")
```

**Example input / output:**

```text
Data Owner -> Defines strategy for Sales data.
Data Steward -> Manages metadata standards and policies, ensuring real-time validation.
Data Custodian -> Maintains accessibility through two-factor authentication while using encrypted cloud storage solutions.
```

**Data:**

```json
{
  "dataOwner": {
    "business_domain": "Sales",
    "strategy": "Maximize customer lifetime value"
  },
  "steward": {
    "metadata_standards": ["ISO 8061", "ISO/IEC 11179"],
    "quality_policies": ["Real-time data validation", "Regular audits"]
  },
  "custodian": {
    "access_control": ["Two-factor authentication", "Least privilege principle"],
    "storage_solutions": ["Cloud-based storage with encryption", "On-premises secure servers"]
  }
}
```

**Flow:**

1. Data Owner
2. Define Strategy
3. Establish Business Context
4. Manage Metadata
5. Set Standards and Policies
6. Ensure Data Quality
7. Implement Validation Rules
8. Maintain Accessibility
9. Apply Access ControlMeasures
10. Physical Storage & Security
11. Use Encrypted Cloud Solutions

