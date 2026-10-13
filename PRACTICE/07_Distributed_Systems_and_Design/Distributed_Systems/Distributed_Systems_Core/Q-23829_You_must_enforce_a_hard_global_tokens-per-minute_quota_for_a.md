---
type: "practice"
difficulty: "Deep"
---

# You must enforce a hard global tokens-per-minute quota for an LLM tenant across regional gateways while tolerating a region partition. Explain why perfect global enforcement and partition tolerance conflict and where you'd land between over- and under-admitting.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a hard global tokens-per-minute (TPM) quota for an LLM tenant across regional gateways while tolerating region partitions, we face a fundamental trade-off between perfect global enforcement and partition tolerance. The goal is to ensure that no more than the allowed number of TPMs are consumed globally while maintaining service availability during network splits.

Firstly, let’s restate the question: We need to design a system that can enforce a global TPM quota without failing requests when regions go offline or become unreachable due to partitions. This requires understanding both the constraints and trade-offs involved.

#### Triage

Key Constraints:

Global Quota Enforcement: Ensure no tenant exceeds its allocated TPM limit globally.
Partition Tolerance: Maintain availability of services during network partitions without sacrificing quota enforcement.

#### Assumptions

There are multiple regional gateways serving the LLM tenant.
Network partitions can occur, making some regions temporarily inaccessible.
The system must handle high throughput and rapid state changes due to partition events.
Step-by-Step Solution

Centralized Quota Enforcement:

A straightforward approach would be to have a single authoritative service that tracks the global TPM usage and enforces the quota centrally. However, this introduces a single point of failure (SPOF) which can disrupt services during partition events.

Decentralized Quota Tracking:

We could distribute quota tracking across regional gateways, but this leads to potential over-admission or under-admission issues due to network partitions.

Hybrid Approach with Eventual Consistency:

Implement a hybrid model where each gateway has its local state and periodically synchronizes with a central service to enforce eventual consistency. This approach balances between global enforcement and partition tolerance.

Event Handling for Partitions:

During partitions, ensure that requests are processed locally until synchronization can resume. Use backoff mechanisms to handle retries in case of temporary network issues.
Detailed Solution

Local State Management:

Each regional gateway maintains a local state tracking the TPM usage.

Centralized Synchronization Service:

Deploy a central service (e.g., Kafka, Raft) that periodically aggregates and enforces the global quota.
When regions become partitioned, they continue to operate based on their last known state.

Request Handling:

Gateway receives a request from an LLM tenant.
Local state is checked for available TPMs.
If sufficient TPMs are available locally, the request is processed; otherwise, it's queued or retried after local state update.

Synchronization Mechanism:

Periodic updates from gateways to the central service ensure eventual consistency.
During partitions, the system operates in a degraded mode but maintains availability.

#### Closing Summary

To balance between perfect global enforcement and partition tolerance, we adopt a hybrid approach where each regional gateway tracks its local state while periodically synchronizing with a central service. This ensures that during network partitions, requests are still processed locally to maintain service availability, albeit possibly at the cost of some degree of over-admission or under-admission until synchronization is resumed.

#### Common Mistakes
and Follow-Ups
Mistake 1: Over-engineering by trying to enforce perfect global state consistency without any fallback mechanisms.
Mistake 2: Ignoring partition events entirely, leading to complete failure during network splits.

#### Follow-Up
Discuss specific tools or patterns like distributed consensus algorithms (e.g., Raft), state replication strategies, and advanced queuing systems that could be employed.

**Comparison:**

```
Region	Max TPM Allowed	Current TPM Usage
R1	500	300
R2	500	400
Request	Region ID	TPM Needed	Result
R1	300	Processed
R2	400	Queued/Retry
```

**Code:**

```python
# Example of local state management in a gateway
class GatewayState:
    def __init__(self):
        self.tpm_usage = 0

    def consume_tpm(self, tpm_needed):
        if self.tpm_usage + tpm_needed > self.max_tpm_allowed:
            return False
        self.tpm_usage += tpm_needed
        return True

# Example of synchronization with a central service
class CentralService:
    def __init__(self):
        self.global_state = {}

    def update_global_state(self, region_id, tpm_usage):
        if region_id in self.global_state:
            self.global_state[region_id] += tpm_usage
        else:
            self.global_state[region_id] = tpm_usage

# Gateway request handling
def handle_request(region_id, tpm_needed):
    gateway = GatewayState()
    central_service = CentralService()

    if gateway.consume_tpm(tpm_needed):
        # Process the request
        print(f"Request processed by {region_id}")
    else:
        print(f"Not enough TPMs available in {region_id}. Queuing or retrying.")

    # Periodically sync with central service
    central_service.update_global_state(region_id, gateway.tpm_usage)
```

**Example input / output:**

```text
Input:
- Region R1 requests 300 TPMs.
- Region R2 requests 400 TPMs.

Output:
Region R1: Request processed. (Total TPM usage in R1 = 300)
Region R2: Not enough TPMs available. Queuing or retrying after local state update.

Periodic sync with central service updates global state:
Central Service: { "R1": 300, "R2": 400 }
```

**Data:**

```json
{
  "regions": [
    {"id": "R1", "max_tpm_allowed": 500},
    {"id": "R2", "max_tpm_allowed": 500}
  ],
  "requests": [
    {"region_id": "R1", "tpm_needed": 300},
    {"region_id": "R2", "tpm_needed": 400}
  ]
}
```

**Flow:**

1. TPMs available?
2. Not enough?
3. Gateway Receives Request
4. Check Local State
5. Process Request
6. Queue or Retry
7. Update Local State & Syncwith Central Service
8. Local State Update & Resyncwith Central Service

