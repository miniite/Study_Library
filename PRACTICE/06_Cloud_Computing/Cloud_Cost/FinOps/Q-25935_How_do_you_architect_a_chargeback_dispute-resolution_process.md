---
type: "practice"
difficulty: "Deep"
---

# How do you architect a chargeback dispute-resolution process? When a team claims the bill they were charged is wrong (shared-cost split, mis-tagging, another team's traffic), what's the audit trail and adjudication mechanism you'd build?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shared_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting a chargeback dispute-resolution process, we're addressing the challenge of ensuring fair billing in shared cloud environments while providing an effective mechanism to resolve disputes when teams believe they've been overcharged or undercharged. The key components include a robust audit trail and an adjudication mechanism that leverages automation for efficiency and transparency.

#### Triaging the Question

The core question revolves around building a system where:

Audit Trail: Detailed logs and records are maintained to trace cost allocations and billing decisions.
Adjudication Mechanism: A process that allows teams to challenge bills and get them resolved with final decisions from authorized personnel.

We need to consider several assumptions:

The cloud environment supports detailed tagging and cost allocation.
Teams have the ability to initiate disputes based on specific criteria (e.g., tags, usage patterns).
There are defined roles for approving or rejecting dispute resolutions.

#### First-Principles Solution

The first step is to design an audit trail that captures all relevant data points needed to validate billing decisions. This includes:

Detailed Cost Records: Logs of cost events with metadata like timestamps, resource IDs, and tags.
Usage Patterns: Historical usage patterns for each team.
Billing Decisions: Automated rules and manual overrides used in billing.

Next, we build a dispute resolution flow that ensures transparency and fairness:

Dispute Initiation: Teams can file disputes based on specific criteria.
Audit Tracing: Systems automatically gather relevant data to support the dispute.
Adjudication Workflow: A workflow system handles the review process with roles for different stakeholders.
Decision Finalization: Automated or manual final decisions are recorded and communicated.
Detailed Breakdown

Audit Trail Design:

Cost Records: Use a cost management service (like AWS Cost Explorer) to capture detailed cost events.
YAML
cost_records:
  - timestamp: 2023-10-01T14:00Z
    resource_id: i-1234567890abcdef
    tag: teamA
    usage: 1GB
    cost: $0.10
Usage Patterns: Maintain historical data to identify normal usage patterns.
JSON
usage_patterns:
  - date_range: [2023-10-01, 2023-10-31]
    team: teamA
    average_usage: 5GB

Dispute Initiation Mechanism:

Teams can submit disputes via a web portal or API.
JSON
dispute_request:
  timestamp: 2023-10-02T16:00Z
  team: teamB
  resource_id: i-9876543210fedcba
  reason: Mis-tagging
  tags: [teamA]

Audit Tracing System:

Automatically gather relevant cost records and usage patterns.
PYTHON
def trace_dispute(dispute_request):
    cost_records = get_cost_records(dispute_request['timestamp'], dispute_request['resource_id'])
    usage_patterns = get_usage_patterns(dispute_request['team'])
    return (cost_records, usage_patterns)

Adjudication Workflow:

Define roles and permissions for different stakeholders.
Yes
No
Dispute Initiation
Is Dispute Valid?
Automatic Resolution
Manual Review
Decision Finalization
Implement a workflow engine to handle the review process.
PYTHON
def resolve_dispute(dispute_request):
    if automatic_resolution_possible(dispute_request['tags'], usage_patterns, cost_records):
        return "Automatic Resolution"
    else:
        return manual_review(dispute_request)

Decision Finalization:

Record and communicate the final decision.
JSON
dispute_response:
  timestamp: 2023-10-05T14:00Z
  team: teamB
  resource_id: i-9876543210fedcba
  resolution: Corrected tagging issue
  status: Approved

#### Closing Summary
and Common Mistakes/Follow-Ups

The chargeback dispute-resolution process is critical for maintaining trust and transparency in shared cloud environments. The key elements are robust audit trails, automated tracing mechanisms, and a well-defined adjudication workflow. Common mistakes include insufficient logging, lack of role-based access controls, and delayed decision-making processes.

For follow-ups, consider scalability issues (handling large numbers of disputes), integration with cost optimization tools, and ensuring compliance with internal policies and external regulations.

**Comparison:**

| Step | Action            | Tool/Service          |
|------|-------------------|-----------------------|
| 1    | Gather Cost Records| AWS Cost Explorer     |
| 2    | Gather Usage Patterns| Custom Database       |
| 3    | Trace Dispute      | Python Script         |
| 4    | Resolve Dispute    | Workflow Engine       |
| 5    | Record Decision    | Logging Service       |

**Code:**

```python
def trace_dispute(dispute_request):
    # Gather relevant data for dispute resolution
    cost_records = get_cost_records(dispute_request['timestamp'], dispute_request['resource_id'])
    usage_patterns = get_usage_patterns(dispute_request['team'])
    return (cost_records, usage_patterns)

def resolve_dispute(dispute_request):
    if automatic_resolution_possible(dispute_request['tags'], usage_patterns, cost_records):
        return "Automatic Resolution"
    else:
        return manual_review(dispute_request)
```

**Example input / output:**

```text
# Dispute Request Example
dispute_request:
  timestamp: 2023-10-02T16:00Z
  team: teamB
  resource_id: i-9876543210fedcba
  reason: Mis-tagging
  tags: [teamA]

# Dispute Response Example
dispute_response:
  timestamp: 2023-10-05T14:00Z
  team: teamB
  resource_id: i-9876543210fedcba
  resolution: Corrected tagging issue
  status: Approved
```

**Data:**

```yaml
# Example Cost Records
cost_records:
  - timestamp: 2023-10-01T14:00Z
    resource_id: i-1234567890abcdef
    tag: teamA
    usage: 1GB
    cost: $0.10

# Example Usage Patterns
usage_patterns:
  - date_range: [2023-10-01, 2023-10-31]
    team: teamA
    average_usage: 5GB
```

**Flow:**

1. Yes
2. No
3. Dispute Initiation
4. Is Dispute Valid?
5. Automatic Resolution
6. Manual Review
7. Decision Finalization

