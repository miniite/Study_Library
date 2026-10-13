---
type: "practice"
difficulty: "Deep"
---

# When two teams disagree on whether a value is 'bad data' or 'a legitimate edge case', how does your framework arbitrate and where does that decision get encoded?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality SLAs and Rule Testing](<../../../../02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a complex, multi-team environment, ensuring data quality often involves navigating disagreements between teams about whether certain values should be classified as "bad data" or accepted as legitimate edge cases. My framework addresses this by establishing clear, objective criteria and decision-making processes that prioritize consistency and accuracy while allowing for the flexibility to handle genuine edge cases.

To triage this question, we need to understand that the core issue is resolving disputes between teams about data quality standards. The arbitration process must be transparent, robust, and capable of scaling across multiple teams. We'll start by defining what constitutes "bad data" versus an "edge case," then outline how these definitions are applied and enforced.

#### Triage

The key elements to consider are:

Definitions: Clearly define what is considered "bad data" versus a legitimate edge case.
Criteria: Establish objective criteria for identifying both bad data and edge cases.
Process: Implement a decision-making process that ensures fairness and consistency.
Documentation: Maintain detailed documentation of decisions and their rationale.

#### First-Principles Solution
Step 1: Define Bad Data and Edge Cases

Bad data typically includes values that are incorrect, inconsistent, or outside expected ranges. An edge case is a value that, while unusual, falls within acceptable operational boundaries.

Step 2: Establish Criteria

We can use statistical measures like Z-scores or IQR to identify outliers indicative of bad data. For edge cases, we define specific thresholds and patterns that allow such values but still ensure overall data quality.

PYTHON
def is_bad_data(value, mean, std_dev):
    z_score = (value - mean) / std_dev
    return abs(z_score) > 3

def is_edge_case(value, threshold):
    return value < threshold
Step 3: Implement Decision-Making Process

To arbitrate disputes, we establish a tiered review process:

Initial Check: A data quality tool automatically flags potential bad data.
Team Review: The team that provided the data reviews flagged values to confirm or override initial checks.
Final Arbitration: If teams disagree, an independent arbiter (e.g., a cross-functional team) makes the final decision based on predefined criteria.
Step 4: Encode Decisions

Decisions are encoded in our Data Quality Management System (DQMS), which logs all actions and provides detailed explanations for each flagged value. This ensures transparency and traceability.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, our framework resolves disputes through a combination of statistical analysis, automated checks, team reviews, and independent arbitration. By documenting decisions clearly, we ensure consistency and reduce misunderstandings.

#### Common mistakes
include:

Over-reliance on Automated Checks: Failing to consider context-specific edge cases.
Lack of Transparency: Not logging or explaining decisions thoroughly.
Ignoring User Feedback: Allowing teams to override automated checks without justification.

#### Follow-ups
might involve refining the statistical models, integrating more advanced machine learning techniques for better accuracy, and continuously evaluating the effectiveness of our framework through A/B testing with different teams.

**Code:**

```python
# Example code snippet
def is_bad_data(value, mean, std_dev):
    z_score = (value - mean) / std_dev
    return abs(z_score) > 3

def is_edge_case(value, threshold):
    return value < threshold
```

**Example input / output:**

```text
Input:
- Value: 1000
- Mean: 500
- Std Dev: 200

Output:
- Bad Data? True (Z-score = 2.5, >3)
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Initial Check
8. Is Value an Outlier?
9. Log as Bad Data
10. Check if Edge Case
11. Log as Edge Case
12. Ambiguous - Submit for TeamReview
13. Team Reviews and Consensus
14. Disagree?
15. Independent Arbiter Decision
16. Final Decision Documented inDQMS
17. Decision Documented inDQMS

