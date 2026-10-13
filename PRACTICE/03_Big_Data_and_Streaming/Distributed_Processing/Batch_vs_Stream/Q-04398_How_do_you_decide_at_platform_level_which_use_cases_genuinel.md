---
type: "practice"
difficulty: "Deep"
---

# How do you decide, at platform level, which use cases genuinely need sub-second freshness versus which teams just say they do? Give me the framework you'd enforce.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data infrastructure that supports both batch and stream processing, one of the critical decisions is determining which use cases genuinely require sub-second freshness versus those where batch updates are acceptable. As an architect/principal engineer in charge of platform-level design, my framework for making this decision involves understanding the business context, evaluating performance requirements, assessing technical feasibility, and balancing cost against benefit.

This process begins with a thorough analysis of each team's data processing needs to determine their true freshness requirements. This is not just about saying they need sub-second updates but about understanding why. For instance, real-time analytics for fraud detection might genuinely require millisecond-level accuracy, whereas financial reporting can accept delays in the order of minutes.

First, we categorize use cases into three tiers: critical, important, and non-critical based on their business impact and urgency. Critical use cases would include scenarios like online transaction systems or real-time bidding where any delay could have significant financial or operational consequences. Important use cases might be things like recommendation engines that need to adapt quickly but can handle slight delays. Non-critical use cases include batch jobs for historical analysis and reporting.

Next, we evaluate the technical feasibility by examining the infrastructure's current capabilities and constraints. This includes assessing whether the existing stream processing framework can support the required freshness or if new technology needs to be introduced. We also consider the cost implications of implementing real-time systems versus batch ones, especially in terms of hardware requirements, network bandwidth, and operational complexity.

**Comparison:**

```
Use Case	Expected Freshness	Categorization
Fraud Detection	Sub-second	Critical
Recommendation Engine	Seconds	Important
Historical Reporting	Minutes	Non-Critical
```

**Code:**

```python
# Example pseudo-code for categorizing use cases
def categorize_use_cases(teams):
    critical = []
    important = []
    non_critical = []

    for team in teams:
        if team['use_case'] == 'fraud_detection':
            critical.append(team)
        elif team['use_case'] == 'recommendation_engine':
            important.append(team)
        else:
            non_critical.append(team)

    return {'critical': critical, 'important': important, 'non_critical': non_critical}
```

**Example input / output:**

```

Given the input data:

YAML
Copy
teams:
  - name: FraudDetectionTeam
    use_case: fraud_detection
    expected_freshness: sub-second
  - name: RecommendationEngineTeam
    use_case: recommendation_engine
    expected_freshness: seconds
  - name: HistoricalAnalysisTeam
    use_case: historical_reporting
    expected_freshness: minutes

The output would categorize the teams as:

YAML
Copy
use_cases:
  critical:
    - FraudDetectionTeam
  important:
    - RecommendationEngineTeam
  non_critical:
    - HistoricalAnalysisTeam
```

**Data:**

```yaml
teams:
  - name: FraudDetectionTeam
    use_case: fraud_detection
    expected_freshness: sub-second
  - name: RecommendationEngineTeam
    use_case: recommendation_engine
    expected_freshness: seconds
  - name: HistoricalAnalysisTeam
    use_case: historical_reporting
    expected_freshness: minutes
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is the use case critical?
7. Classify as Critical
8. Is the use case important?
9. Classify as Important
10. Classify as Non-Critical
11. Final Classification: Critical
12. Final Classification: Important
13. Final Classification:Non-Critical

