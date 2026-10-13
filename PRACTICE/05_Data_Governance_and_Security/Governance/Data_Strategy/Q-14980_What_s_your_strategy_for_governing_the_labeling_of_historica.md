---
type: "practice"
difficulty: "Deep"
---

# What's your strategy for governing the labeling of historical incidents (root cause, severity, blast radius) so the labels are consistent enough to train causal models?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Incident Data Reuse](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Incident_Data_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern the labeling of historical incidents in a way that ensures consistency and reliability for training causal models, we need to establish a comprehensive strategy involving data quality, metadata management, and governance processes. This involves defining clear standards, leveraging automation where possible, and continuously monitoring and improving the labeling process.

#### Triaging the Question

The core of this question is about ensuring the accuracy and consistency of incident labels (root cause, severity, blast radius) in historical data so that they can be effectively used to train machine learning models. The interviewer is probing your understanding of how to manage and govern a large dataset, especially one involving critical business operations.

#### Assumptions
Historical Incident Data: We have existing incident records with various attributes like root cause, severity, blast radius.
Machine Learning Models: These will be used for predictive maintenance or anomaly detection in the future.

#### Cons
istency Requirements: The labels need to be consistent across all incidents within a certain timeframe and granularity.

#### Solution

Define Labeling Standards

Root Cause: Define categories such as software bugs, hardware failures, network issues, user errors, etc.
Severity: Classify into critical (down time), high (significant impact), medium (moderate impact), low (minor).
Blast Radius: Indicate the affected systems or users.

Labeling Process

Manual Labeling: Initially, manual labeling might be required to ensure accuracy.
Automated Tagging: Implement rules and heuristics to automatically tag incidents based on predefined criteria.
Review and QA: Set up a quality assurance process where manually labeled data can be reviewed by domain experts.

Metadata Management

Version Control: Ensure metadata schema is version-controlled to manage changes over time.
Lineage Tracking: Maintain lineage records of how labels were assigned, including the rationale behind each label.

Continuous Improvement

Feedback Loops: Implement mechanisms for feedback from model performance to refine labeling standards.
Iterative Refinement: Periodically reassess and update labeling rules based on new data or changing business needs.

Governance Framework

Stakeholder Involvement: Engage key stakeholders (IT, Ops, Business Analysts) in the governance process.
Compliance Checks: Ensure the labeling practices comply with any regulatory requirements.

#### Closing Summary
and Common Mistakes

To ensure consistent incident labels, a multi-faceted approach is necessary. It involves defining clear standards, implementing structured processes for both manual and automated tagging, maintaining robust metadata management, and continuously improving based on feedback. Common mistakes include failing to define clear labeling standards at the outset, neglecting quality assurance steps, and not integrating governance practices.

#### Follow-Ups
Automation Levels: How would you balance automation with human oversight?
Label Drift: What measures can be taken to address label drift over time?
Impact on Model Performance: How does inconsistent labeling impact model training?

**Comparison:**

```
Incident ID	Root Cause	Severity	Blast Radius
12345	software bug	critical	system C
67890	hardware failure	high	system A, system B
```

**Code:**

```python
# Example Python code for automated tagging based on heuristics
def tag_severity(log):
    if 'server' in log and 'crash' in log:
        return 'critical'
    elif 'user' in log:
        return 'low'
    else:
        return 'medium'

# Sample incident log
log = "Server crash detected by monitoring system"
severity_label = tag_severity(log)
print(f"Severity: {severity_label}")
```

**Example input / output:**

```
Incident ID	Root Cause	Severity	Blast Radius
12345	network issue	medium	system A, system B
Incident ID	Root Cause	Severity	Blast Radius
67890	software bug	critical	system C
PLAINTEXT
Copy
Incident: 12345
Root Cause: network issue
Severity: medium
Blast Radius: [system A, system B]
```

**Data:**

```json
{
  "incident": {
    "id": "12345",
    "rootCause": "network issue",
    "severity": "medium",
    "blastRadius": ["system A", "system B"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Incident Log
4. Is Label Required?
5. Apply Heuristics
6. Leverage Metadata
7. Review & QA
8. Save Labels
9. Use Existing Labels
10. Save Labels

