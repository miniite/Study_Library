---
type: "practice"
difficulty: "Deep"
---

# You're the principal on a platform where teams keep choosing last-write-wins for convenience and silently losing data under clock skew. Design the guardrails, defaults, and review gates that make LWW a deliberate, justified choice rather than an accidental one.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Federated vs Centralized Data Ownership](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Federated_vs_Centralized_Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of teams choosing last-write-wins (LWW) in distributed systems without proper awareness of its pitfalls, we need to design robust guardrails and defaults that ensure LWW is a deliberate choice rather than an accidental one. This approach involves defining clear guidelines, integrating automated checks, and establishing comprehensive review processes.

The core challenge here is ensuring data consistency while avoiding the risks associated with LWW, such as data loss under clock skew or race conditions. We'll start by outlining the key principles and then detail specific measures to enforce these principles.

Understand LWW: First, we must define what LWW means in our context: it's a strategy where the last write operation overwrites any previous writes. While this can simplify operations, it inherently leads to data loss if multiple writes occur simultaneously and not all are detected by the system.

Risk Assessment: We need to assess the risks associated with LWW:

Data loss under clock skew: If two writes happen in different time zones or have significant network latency, one might be seen as the last write before the other.
Race conditions: If multiple writes occur very close in time and not all are captured by the system, data integrity can be compromised.

Guardrails and Defaults: We'll implement guardrails to restrict LWW use and defaults that promote safer practices:

Default Consistency Model: By default, use a higher consistency model like strong eventual consistency or session consistency.
Explicit LWW Markers: Ensure all operations using LWW are explicitly marked as such, making it clear which ones are exceptions.

Automated Checks and Validation: Integrate automated checks to detect potential issues:

Clock Skew Detection: Implement a mechanism to detect clock skew between nodes in the distributed system.

#### Cons
istency Checkers: Use consistency check tools or libraries that can verify data integrity during writes.

Review Gates: Establish review processes to ensure every LWW operation is justified:

Code Reviews: Ensure all code changes involving LWW are thoroughly reviewed by a designated team.
Design Documents: Require detailed design documents explaining the rationale for using LWW, including expected failure modes and mitigation strategies.

Training and Documentation: Provide training sessions and comprehensive documentation to educate teams on the risks of LWW and best practices:

Workshops: Conduct regular workshops or webinars focusing on distributed systems consistency models.
Guidelines Document: Publish a detailed guidelines document that outlines when and how LWW can be used safely.

Monitoring and Logging: Implement monitoring and logging to track the usage of LWW and detect potential issues early:

Metrics Collection: Collect metrics on LWW operations, such as frequency and impact.
Incident Response: Have clear incident response protocols in place for handling any data inconsistencies arising from LWW.

**Comparison:**

```
Review Gate Stage	Description
Code Reviews	Ensures all changes are reviewed by a designated team.
Design Documents	Requires detailed explanations of the use case for LWW.
```

**Code:**

```python
# Example of an automated clock skew detection function
def detect_clock_skew(node1_time, node2_time):
    threshold = 0.5  # seconds
    if abs(node1_time - node2_time) > threshold:
        return True
    return False

# Example of a consistency checker that logs potential issues
class ConsistencyChecker:
    def check_write(self, write_op):
        if not self.detect_clock_skew(write_op.node_times[0], write_op.node_times[1]):
            logging.warning("Potential clock skew detected in LWW operation")
        # Further checks and validations
```

**Example input / output:**

```text
Input:
- Write operation with node times: [1640995200, 1640995210]
- Clock skew threshold: 0.5 seconds

Output:
- Log message: "Potential clock skew detected in LWW operation"
```

**Data:**

```json
{
  "write_op": {
    "id": "123",
    "node_times": [1640995200, 1640995210], // Unix timestamps in seconds
    "data": {"key": "value"}
  }
}
```

**Flow:**

1. Yes
2. No
3. Pass
4. Fail
5. Check Write Operation
6. Detect Clock Skew?
7. Log Warning
8. Perform Consistency Check
9. Write to DB
10. Handle Inconsistency

