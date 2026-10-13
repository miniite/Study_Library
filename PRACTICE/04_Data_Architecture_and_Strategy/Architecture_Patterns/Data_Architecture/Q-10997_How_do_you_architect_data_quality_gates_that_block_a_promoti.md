---
type: "practice"
difficulty: "Deep"
---

# How do you architect data quality gates that block a promotion from silver to gold, and design them so a false-positive gate failure at 3am doesn't stall the entire estate? What circuit-breaker, override, and paged-owner model keeps quality enforcement from becoming an availability risk?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Layered Pipelines Bronze Silver Gold](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When architecting data quality gates to prevent promotions from silver to gold status, it's crucial to ensure that false-positive gate failures are isolated and do not stall the entire estate. This requires a robust circuit-breaker mechanism, an override process, and a paged-owner model for handling quality enforcement issues efficiently.

#### Triage

The core of this question is about designing a data quality gate system in a way that prevents cascading failures while ensuring high-quality standards are maintained. Specifically, the challenge is to handle a scenario where a false-positive quality check fails at 3 AM and how to mitigate the risk this poses without impacting overall availability.

First Principles

Define the Quality Gates: Establish clear criteria for moving data from silver to gold status. These can include completeness checks, consistency rules, accuracy validations, and other domain-specific requirements.

Circuit-Breaker Mechanism: Implement a circuit breaker that isolates a failing quality gate early, preventing it from causing a full system failure.

Override Process: Provide a mechanism to manually approve or override the fail state of a specific data batch without affecting ongoing operations.

Paged-Owner Model: Ensure that each piece of data has a responsible owner who can address issues promptly and escalate them if necessary.

Detailed Solution
1. Defining Quality Gates

First, we need to define what constitutes "gold-quality" data. This involves setting thresholds for various quality dimensions such as:

Completeness (all mandatory fields are filled)

#### Cons
istency (data matches across multiple sources)
Accuracy (data meets business rules and standards)

For example:

SQL
CREATE TABLE QualityGates (
    id INT PRIMARY KEY,
    batch_id VARCHAR(50),
    completeness BOOLEAN,

#### cons
istency BOOLEAN,
    accuracy BOOLEAN,
    timestamp TIMESTAMP,
    status VARCHAR(20) -- e.g., "pass", "fail"
);
2. Circuit-Breaker Mechanism

A circuit breaker is a protective mechanism that can be used to prevent a single point of failure from bringing down the entire system. When a quality check fails, it triggers a circuit breaker that temporarily disables further checks on that data batch.

PYTHON
class QualityCircuitBreaker:
    def __init__(self):
        self.failure_threshold = 3  # Number of consecutive failures before tripping
        self.current_failures = 0

    def evaluate_quality(self, quality_gate_result: dict) -> bool:
        if not all(quality_gate_result.values()):
            self.current_failures += 1
            return False
        else:
            self.current_failures = 0
            return True

    def trip_circuit(self):
        if self.current_failures >= self.failure_threshold:
            print("Circuit breaker tripped. Suspending further quality checks.")
            # Implement logic to suspend the failed batch or mark it as suspicious
3. Override Process

Provide a mechanism for authorized users to manually approve batches that fail quality checks due to known issues, ensuring data can still be promoted.

SQL
CREATE TABLE QualityOverride (
    id INT PRIMARY KEY,
    batch_id VARCHAR(50),
    user_name VARCHAR(100),
    reason TEXT,
    timestamp TIMESTAMP
);

An override request could look like:

A system administrator can enter a manual approval through an interface.
The approval is recorded in the QualityOverride table and can be reviewed later for consistency.
4. Paged-Owner Model

Each data batch should have a clear owner responsible for addressing quality issues promptly. This ensures that no single failure goes unnoticed or unaddressed.

SQL
CREATE TABLE DataBatchOwners (
    id INT PRIMARY KEY,
    batch_id VARCHAR(50),
    owner_name VARCHAR(100),
    contact_email VARCHAR(255)
);

In practice, the data batch owners can be notified via email or a mobile app when a quality check fails. They are responsible for investigating and resolving issues.

#### Closing Summary

By implementing a circuit breaker to isolate failing batches, providing an override process for manual approvals, and ensuring each batch has a clear owner, we mitigate the risk of false-positive failures causing cascading system failures. This approach maintains high-quality standards while keeping the overall availability of the data promotion pipeline intact.

#### Common Mistakes/Follow-Ups
False Positives: Ensure that quality checks are comprehensive but not overly stringent to avoid triggering false positives.
Performance Impact: Monitor the performance impact of circuit breaker trips and ensure they do not degrade system responsiveness.
User Training: Provide training for administrators and batch owners on how to use override mechanisms effectively.

**Comparison:**

| 1 | B123 | True | False | True | 2023-09-01T03:00:00Z | fail
| 1 | B123 | admin | Known issue with consistency check | 2023-09-01T04:00:00Z
| 1 | B123 | Jane Doe | jdoe@example.com

**Code:**

```python
class QualityCircuitBreaker:
    def __init__(self):
        self.failure_threshold = 3
        self.current_failures = 0

    def evaluate_quality(self, quality_gate_result: dict) -> bool:
        if not all(quality_gate_result.values()):
            self.current_failures += 1
            return False
        else:
            self.current_failures = 0
            return True

    def trip_circuit(self):
        if self.current_failures >= self.failure_threshold:
            print("Circuit breaker tripped. Suspending further quality checks.")
```

**Example input / output:**

```

Example input and output for the circuit breaker:

PYTHON
Copy
quality_gates = [
    {"batch_id": "B123", "completeness": True, "consistency": False, "accuracy": True},
    {"batch_id": "B456", "completeness": True, "consistency": True, "accuracy": False}
]

breaker = QualityCircuitBreaker()
for gate in quality_gates:
    if not breaker.evaluate_quality(gate):
        print(f"Quality check failed for batch {gate['batch_id']}")
        breaker.trip_circuit()
```

**Data:**

```sql
CREATE TABLE QualityGates (
    id INT PRIMARY KEY,
    batch_id VARCHAR(50),
    completeness BOOLEAN,
    consistency BOOLEAN,
    accuracy BOOLEAN,
    timestamp TIMESTAMP,
    status VARCHAR(20)
);

CREATE TABLE QualityOverride (
    id INT PRIMARY KEY,
    batch_id VARCHAR(50),
    user_name VARCHAR(100),
    reason TEXT,
    timestamp TIMESTAMP
);

CREATE TABLE DataBatchOwners (
    id INT PRIMARY KEY,
    batch_id VARCHAR(50),
    owner_name VARCHAR(100),
    contact_email VARCHAR(255)
);
```

**Flow:**

1. Yes
2. No
3. Fail Count < Threshold
4. Fail Count >= Threshold
5. Yes
6. No
7. Start
8. Quality Check Pass?
9. Log Passed
10. Circuit Breaker Eval.
11. Continue Checks
12. Circuit Tripped, Notify Owner
13. Override Requested?
14. Manual Override Approved
15. Alert Admins, Monitor Issues
16. Next Batch

