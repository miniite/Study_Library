---
type: "practice"
difficulty: "Expert"
---

# Your streaming platform (Kafka/Kinesis) and your lake are owned by different teams with conflicting retention and schema-evolution policies. As principal, how do you architect the boundary and resolve the org conflict?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse Vendor Exit Strategy](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_Vendor_Exit_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a backend architect tasked with resolving the organizational conflicts around data architecture for a streaming platform using Kafka/Kinesis and a lake environment, the key challenge is to create a cohesive design that aligns both teams' goals while respecting their autonomy. The goal is not just to stitch together systems but also to foster collaboration and ensure that long-term data integrity and usability are prioritized.

#### Triaging the Question

The core issue here involves two distinct organizational entities with different requirements:

Streaming Platform Team: Likely focused on low-latency, real-time processing and rapid ingestion of data.
Lake Team: May prioritize long-term storage, analytics, and historical data analysis.

Both teams have conflicting policies in terms of schema evolution and data retention. The streaming platform might need immediate access to the latest data, while the lake team could require preserving data for months or even years. Additionally, changes to schemas must be handled carefully to avoid breaking existing processes and applications.

#### Assumptions
Data Streams: Both teams are dealing with large volumes of high-speed data.
Schema Evolution: The platform requires schema flexibility to accommodate rapid changes.
Retention Policies: Different retention requirements exist, necessitating a nuanced approach.

#### First-Principles Solution
1. Define Clear Interfaces

Define clear interfaces between the streaming platform and the lake system using well-defined API boundaries. This will help in managing data transfer without tight coupling.

Streaming Platform
Real-time Data Interface
Lake System
Lake Storage & Analytics
2. Schema Evolution Strategy

Implement a robust schema evolution strategy that can handle changes in both systems without causing disruptions. This could involve:

Versioned Schemas: Use version numbers for schemas to track changes.
Incremental Updates: Implement incremental updates where only necessary fields change.
3. Retention Policies Management

Establish flexible retention policies that allow different data lifecycles while ensuring compliance with both teams’ needs. This might require tiered storage solutions:

Hot Data
Lake Storage
Warm Data
Cold Archive
4. Conflict Resolution Mechanism

Create a conflict resolution mechanism where both teams can negotiate changes in their respective policies. This could involve regular meetings to discuss and align on data usage and storage requirements.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The architecture needs to balance the operational demands of the streaming platform with the long-term analytical needs of the lake system. By defining clear interfaces, implementing robust schema evolution strategies, managing tiered retention policies, and establishing a conflict resolution mechanism, we can ensure that both teams' goals are met without compromising on data integrity or usability.

#### Common Mistakes

Ignoring Organizational Boundaries: Failing to recognize the different needs and constraints of each team.
Tight Coupling: Not defining clear interfaces, leading to rigid and inflexible architectures.
Inadequate Schema Management: Overlooking the importance of versioned schemas and incremental updates.

#### Follow-Ups

How would you handle data lineage and governance in this setup?
What role do you see for machine learning models in such a system?
How can we ensure compliance with regulatory requirements?

**Comparison:**

| Storage Type | Retention Period |
|--------------|------------------|
| Hot          | 30 days          |
| Warm         | 180 days         |
| Cold Archive | 5 years          |

**Code:**

```python
# Example of versioned schema handling in Python (simplified)
class SchemaVersion:
    def __init__(self, version, fields):
        self.version = version
        self.fields = fields

def update_schema(current_schema, new_fields):
    if not any(field['name'] == 'new_field' for field in current_schema.fields):
        # Increment version and add new field
        return SchemaVersion(current_schema.version + 1, current_schema.fields + [{'name': 'new_field', 'type': 'string'}])
    else:
        return current_schema

# Example usage
current_schema = SchemaVersion(1, [{'name': 'old_field', 'type': 'integer'}])
updated_schema = update_schema(current_schema, {'name': 'new_field', 'type': 'string'})
```

**Example input / output:**

```text
Input:
- Real-time stream with updated schema (version 3)
- Lake system with three tiers of storage

Output:
- Stream data ingested into hot storage, versioned accordingly
- Historical data moved to appropriate warm or cold archive tier based on schema version and retention policies
```

**Data:**

```json
{
  "data_streams": [
    {
      "id": "stream1",
      "type": "real_time",
      "schema_version": 3,
      "fields": [
        {"name": "old_field", "type": "integer"},
        {"name": "new_field", "type": "string"}
      ]
    },
    {
      "id": "stream2",
      "type": "real_time",
      "schema_version": 2,
      "fields": [
        {"name": "old_field", "type": "integer"}
      ]
    }
  ],
  "lake_policies": [
    {
      "storage_type": "hot",
      "retention_days": 30
    },
    {
      "storage_type": "warm",
      "retention_days": 180
    },
    {
      "storage_type": "cold_archive",
      "retention_years": 5
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Hot
4. Warm
5. Real-time Data
6. Schema Version Check
7. Is Schema Updated?
8. Lake Storage - Hot Tier
9. Incremental Update &Retention Check
10. Hot or Warm Tier?
11. Lake Storage - Warm Tier
12. Cold Archive Tier
13. Lake Analytics

