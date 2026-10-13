---
type: "practice"
difficulty: "Expert"
---

# You must architect how the platform handles a 10x cardinality explosion in a metrics/observability system that's now costing more than the product it monitors — bend the cost curve without going blind.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The goal is to design a scalable observability system that can handle a 10x increase in data cardinality while reducing costs and ensuring minimal impact on operational visibility. The solution involves leveraging distributed tracing, log aggregation, and intelligent sampling techniques, all while optimizing storage and query performance.

#### Triaging the Question

This question asks for an HLD (High-Level Design) to address a significant scalability challenge in a metrics/observability system. Specifically, we need to scale up handling 10x more data without increasing costs disproportionately. The interviewer is probing our ability to balance cost efficiency with operational visibility.

#### Assumptions
and Key Considerations:
Current Costs: The current observability system is excessively costly.
Data Cardinality: We're dealing with a potential 10x increase in data volume.
Operational Impact: Minimal disruption in the monitoring process is required.
Cost Efficiency: Any solution must be cost-effective.

#### First Principles Solution
Step 1: Understanding the Current System

Assuming the current system uses centralized storage and processing, we need to decouple these functions. We'll leverage distributed tracing for detailed event tracking and log aggregation for broad data collection.

Step 2: Decoupling Components
Distributed Tracing: Implement a system using OpenTracing or similar standards.
Log Aggregation: Use tools like ELK stack, Fluentd, or Prometheus for centralized logging.
Step 3: Intelligent Sampling Techniques

To handle the increased cardinality:

Downsampling Logs: Aggregate logs at lower granularity to reduce storage requirements.
Sampling Traces: Randomly sample traces while preserving key events.
Step 4: Storage Optimization
Tiered Storage: Use cheaper storage tiers for older data, like S3 or cold storage solutions.
Indexing and Query Optimization: Employ efficient indexing techniques and query optimization strategies to minimize read costs.
Step 5: Data Sharding and Replication

Distribute the data across multiple nodes to ensure load balancing:

Horizontal Scaling: Use a sharding strategy based on time or log ID.
Replication: Ensure redundancy for high availability while minimizing storage overhead.

#### Implementation
Details:
Data Ingest
Distributed Tracing
Log Aggregation
Tiered Storage
Query Optimization
Replication and Sharding
Code Example for Log Aggregation:
PYTHON
# Sample code snippet for log aggregation using Fluentd
from fluent import sender

def send_log(message):
    with sender(FluentSender(host='localhost', port=24224)) as sender:
        sender.emit('app.logs', {'message': message, 'timestamp': time.time()})

send_log("Application error logged")

#### Closing Summary
and Common Mistakes:

The proposed solution involves decoupling data processing and storage to handle increased cardinality efficiently. Key steps include implementing distributed tracing, intelligent sampling, optimizing storage tiers, and sharding the data for scalability. Common mistakes include overcomplicating the architecture or failing to consider cost optimization techniques.

#### Follow-Ups
What are your thoughts on using a hybrid approach with both centralized and distributed components?
How would you handle edge cases where detailed tracing is critical?
Can you discuss the trade-offs between storage cost and query performance in this scenario?

**Comparison:**

```
Component	Description	Implementation
Tracing	Distributed tracing system	OpenTracing
Logging	Centralized log aggregation	Fluentd
Storage	Tiered storage solution	S3, Cold Storage
```

**Code:**

```python
# Sample code snippet for log aggregation using Fluentd
from fluent import sender

def send_log(message):
    with sender(FluentSender(host='localhost', port=24224)) as sender:
        sender.emit('app.logs', {'message': message, 'timestamp': time.time()})

send_log("Application error logged")
```

**Example input / output:**

```text
Input (Sample Log Entry):
timestamp: "2023-10-15T14:00:00Z"
level: "ERROR"
logger: "user.service"
message: "Failed to process user request"

Output (Aggregated Log Entry):
timestamp: "2023-10-15T14:00:00Z"
level: "ERROR"
logger: "user.service"
message: "Failed to process user request"
```

**Data:**

```yaml
# Sample data schema for log aggregation
log_entry:
  timestamp: string
  level: string
  logger: string
  message: string
  trace_id: string
  span_id: string
```

**Flow:**

1. Data Ingest
2. Distributed Tracing
3. Log Aggregation
4. Tiered Storage
5. Query Optimization
6. Replication and Sharding

