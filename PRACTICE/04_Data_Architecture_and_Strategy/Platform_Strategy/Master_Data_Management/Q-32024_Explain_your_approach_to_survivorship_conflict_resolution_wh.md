---
type: "practice"
difficulty: "Deep"
---

# Explain your approach to survivorship conflict resolution when two authoritative sources disagree on a legally-significant attribute (e.g., tax ID). Do you pick, flag, or escalate — and who is accountable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Records and Survivorship](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Survivorship conflict resolution in Master Data Management (MDM) is critical when two authoritative sources provide different values for a key attribute like a tax ID. The approach should be guided by clear policies and procedures, balancing accuracy with operational efficiency. Here’s an overview of the process, starting from first principles:

TL;DR: When resolving survivorship conflicts, we must determine whether to pick one source over another, flag both sources as conflicting, or escalate for further review. Accountability is crucial, ensuring that changes are tracked and responsibilities assigned.

#### Triage
The question probes how you handle discrepancies in MDM systems where authoritative data from different sources contradict each other on a legally-significant attribute. This involves understanding the business context, defining rules to resolve conflicts, and determining accountability mechanisms.

First Principles Approach:

Identify the Attribute Importance: Tax IDs are typically crucial for legal and compliance reasons. We must ensure that any conflict is resolved correctly.
Define Conflict Resolution Policies: Establish clear policies on how to handle such conflicts. These could include rules like picking the most recent data, a pre-defined priority of sources, or escalating based on attribute importance.
Determine Accountability: Define who should be responsible for making the final decision and ensuring that changes are logged and audited.

Step-by-Step Solution:

Data Collection & Validation:

Gather data from both authoritative sources and validate it to ensure accuracy.

Conflict Detection:

Implement a system to detect conflicts at the time of data ingestion or during regular reconciliation processes.
For instance, using a script that compares tax IDs in real-time.

Policy Application:

Apply predefined policies to resolve the conflict. For example:
PYTHON
def resolve_tax_id_conflict(source1, source2):
    # Example policy: Pick the most recent data
    if source1['timestamp'] > source2['timestamp']:
        return source1
    else:
        return source2
Alternatively, use a predefined priority list:
PYTHON
def resolve_tax_id_conflict(source1, source2):
    # Example priority: 'sourceA' is preferred over 'sourceB'
    if source1['priority'] > source2['priority']:
        return source1
    else:
        return source2

Escalation Mechanism:

If no clear policy resolves the conflict, escalate to a higher authority or team for decision-making.
PYTHON
def escalate_conflict(source1, source2):
    # Send email notification to compliance team
    send_email("Compliance Team", "Tax ID Conflict: Source A vs. Source B")
    return None  # Indicate no clear resolution yet

Logging & Auditing:

Log all decisions and the rationale behind them.
JSON
{
    "decision": "sourceA",
    "reason": "Timestamp indicates more recent data entry.",
    "timestamp": "2023-10-01T14:30:00Z"
}

Review and Improvement:

Periodically review conflict resolution outcomes to improve policies based on actual use cases.
Conduct training sessions for stakeholders on handling specific types of conflicts.

#### Closing Summary
The approach to survivorship conflict resolution in MDM involves a combination of policy-driven decision-making, real-time validation, escalation mechanisms, and thorough logging. Accountability is key to ensuring that changes are properly tracked and justified.

#### Common Mistakes
& Follow-Ups:

Failing to establish clear policies can lead to inconsistent resolutions.
Over-reliance on automated systems without human oversight can miss nuanced decisions.
Not maintaining detailed logs makes it difficult to trace back issues or improvements.

**Comparison:**

```
Timestamp	Source A (Tax ID)	Priority
2023-10-01T14:00:00Z	1234567890	1
2023-10-01T14:15:00Z	9876543210	2
```

**Code:**

```python
def resolve_tax_id_conflict(source1, source2):
    if source1['timestamp'] > source2['timestamp']:
        return source1
    else:
        return source2

def escalate_conflict(source1, source2):
    send_email("Compliance Team", "Tax ID Conflict: Source A vs. Source B")
    return None
```

**Example input / output:**

```text
Input:
{
  "sourceA": {"tax_id": "1234567890", "timestamp": "2023-10-01T14:00:00Z", "priority": 1},
  "sourceB": {"tax_id": "9876543210", "timestamp": "2023-10-01T14:15:00Z", "priority": 2}
}

Output:
{
  "selected_tax_id": "9876543210",
  "decision_reason": "Source B has a more recent timestamp and higher priority."
}
```

**Data:**

```json
{
  "sourceA": {"tax_id": "1234567890", "timestamp": "2023-10-01T14:00:00Z", "priority": 1},
  "sourceB": {"tax_id": "9876543210", "timestamp": "2023-10-01T14:15:00Z", "priority": 2}
}
```

**Flow:**

1. Yes
2. Policy Based
3. Escalate
4. Data Ingestion
5. Detect Conflict?
6. Resolve Conflicts
7. Select Source
8. Escalation Review
9. Log Decision
10. Review & Improve Policies
11. Audit Logs
12. Conflict Resolution

