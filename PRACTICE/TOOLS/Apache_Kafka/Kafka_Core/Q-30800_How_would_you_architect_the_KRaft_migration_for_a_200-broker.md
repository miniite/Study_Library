---
type: "practice"
difficulty: "Expert"
---

# How would you architect the KRaft migration for a 200-broker ZooKeeper cluster with zero customer-visible downtime? Define the phased plan the rollback story per phase and the correlated risks the metadata quorum introduces.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ KRaft Mode](<../../../../TOOLS/Apache_Kafka/Kafka_Core/When_Not_to_Use_Kafka/KRaft_Mode.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect the KRaft migration for a 200-broker ZooKeeper cluster with zero customer-visible downtime, we need to carefully plan out multiple phases that ensure both data consistency and operational integrity while minimizing disruptions. The key steps include:

#### Overview
The primary goal is to migrate from ZooKeeper to KRaft without any service interruptions or visible downtime for customers. This involves a phased approach, ensuring smooth data transfer and validation.

#### Triage

Current State: A 200-broker cluster with existing data and operational dependencies.
Requirements: Zero customer-visible downtime, safe rollback strategies, and minimal risk exposure during the migration.

#### Assumptions
The cluster is currently running KRaft-enabled brokers, and we have the necessary tools and infrastructure for smooth transitions.

Phased Migration Plan:

Phase 1: Preparation
Identify critical dependencies and configurations.
Validate the health of current ZooKeeper clusters.
Ensure all Kafka brokers are KRaft-enabled with proper configuration updates.
Phase 2: Initial Rollout (Selective Brokers)
Migrate a small subset of brokers initially to test the KRaft cluster setup.
Monitor and validate data consistency, performance, and any potential issues.
Phase 3: Full Cluster Migration (Rolling Update)
Gradually migrate additional brokers in smaller groups to ensure continuous service availability.
Implement monitoring and alerting systems for real-time feedback.
Phase 4: Final Validation and Testing
Conduct comprehensive tests with full cluster operations.
Validate data integrity, performance metrics, and customer experience.

Rollback Strategy:

Develop a rollback plan that can revert the KRaft migration back to ZooKeeper if any issues arise.
Ensure all brokers are in a safe state before rolling them back, using automated scripts for consistency checks.

Correlated Risks:

Metadata Quorum: KRaft introduces a new metadata quorum mechanism that requires careful management to ensure data durability and availability.
Data Consistency: Ensuring that all partitions are correctly migrated and no data loss occurs is critical.
Operational Risk: Migrating a large cluster can introduce operational risks if not managed properly, such as network outages or broker downtime.

#### Closing Summary

A phased KRaft migration plan with robust testing and rollback strategies ensures zero customer-visible downtime for a 200-broker cluster.
Key factors include careful preparation, selective initial rollout, gradual full-cluster migration, and comprehensive validation to manage the risks associated with KRaft metadata quorum.

#### Common Mistakes/Follow-Ups

Failing to properly validate data consistency during the transition phase can lead to critical issues.
Overlooking network or infrastructure dependencies that might cause outages.
Not having a well-defined rollback strategy in place can leave operations vulnerable if something goes wrong.

**Comparison:**

```
Phase	Description	Brokers Involved	Health Check	Validation
1	Preparation	All	N/A	N/A
2	Initial Rollout (Selective Brokers)	First 50	Yes	Yes
3	Full Cluster Migration	Remaining 150	Yes	Yes
4	Final Validation	All	Yes	Yes

Note: This table is a simplified representation and should be expanded with actual data during the migration.
```

**Code:**

```python
# Example of a simple health check script for Kafka brokers
def check_kafka_broker_health(broker_id):
    # Placeholder for actual health check logic
    return True  # Assume broker is healthy

# Example of a phased migration function
def migrate_to_kraft(cluster_size, initial_group_size=50):
    current_group = 1
    while current_group <= cluster_size / initial_group_size:
        start_broker = (current_group - 1) * initial_group_size + 1
        end_broker = min(current_group * initial_group_size, cluster_size)
        migrate_brokers(start_broker, end_broker)
        validate_migration(start_broker, end_broker)
        current_group += 1

def migrate_brokers(start_broker, end_broker):
    for broker_id in range(start_broker, end_broker + 1):
        # Logic to enable KRaft on brokers
        pass

def validate_migration(start_broker, end_broker):
    for broker_id in range(start_broker, end_broker + 1):
        if not check_kafka_broker_health(broker_id):
            raise Exception(f"Migration validation failed for broker {broker_id}")
```

**Example input / output:**

```text
| Input | Description            | Output               |
|-------|------------------------|----------------------|
| `check_kafka_broker_health(1)` | Check health of broker 1. | `True` (healthy)     |
| `migrate_to_kraft(200)`       | Migrate 200 brokers to KRaft. | N/A                 |

Note: The output in this table is a placeholder and should be replaced with actual data during the migration process.
```

**Data:**

```json
{
  "cluster_size": 200,
  "initial_group_size": 50,
  "migration_phases": [
    {"phase": 1, "description": "Preparation"},
    {"phase": 2, "description": "Initial Rollout (Selective Brokers)"},
    {"phase": 3, "description": "Full Cluster Migration (Rolling Update)"},
    {"phase": 4, "description": "Final Validation and Testing"}
  ]
}
```

**Flow:**

1. Preparation
2. Initial Rollout
3. Rolling Update
4. Final Validation
5. Rollback

