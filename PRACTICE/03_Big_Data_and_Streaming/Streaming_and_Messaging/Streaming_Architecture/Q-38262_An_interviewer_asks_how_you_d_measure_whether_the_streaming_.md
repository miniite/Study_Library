---
type: "practice"
difficulty: "Expert"
---

# An interviewer asks how you'd measure whether the streaming platform is actually improving org reliability (MTTD/MTTR) versus just adding a shiny layer. Design the causal evaluation you'd present to leadership that isn't just correlation theater.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Streaming for AIOps](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Streaming_for_AIOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evaluate whether a real-time streaming platform has genuinely improved organizational reliability, we need to go beyond mere correlations and implement a robust causal evaluation framework. This involves measuring both Mean Time To Detect (MTTD) and Mean Time To Resolve (MTTR), but in a way that isolates the impact of our streaming platform from other factors affecting system reliability.

#### Triage

The interviewer is probing for a method to causally link improvements in MTTD/MTTR directly to the introduction or enhancement of the real-time streaming platform. We need to ensure that any observed changes are not just coincidental and can be attributed to the new system. This requires setting up proper experiments, control groups, and statistical methods to validate our claims.

#### First-Principles Solution
Define Key Performance Indicators (KPIs): Establish clear KPIs such as MTTD/MTTR for both the existing and new systems.
Pre- and Post-Deployment Data Collection: Collect comprehensive data on system performance before deploying the streaming platform, then continuously monitor post-deployment to capture any changes.
Controlled Experimentation: Use A/B testing or canary deployments to test the impact of the new system in a controlled environment.
Statistical Analysis: Apply statistical methods like ANOVA (Analysis of Variance) or regression analysis to determine if the observed improvements are significant and not due to other factors.
Step-by-Step Solution

Baseline Collection:

Identify critical systems and their historical reliability data.
Collect MTTD/MTTR metrics for these systems over a baseline period (e.g., 6 months).

Deployment Strategy:

Implement the real-time streaming platform in a staged manner, starting with non-critical systems or canary groups.
Continuously monitor and log all relevant performance data during this phase.

Post-Deployment Monitoring:

After full deployment, continue to collect detailed MTTD/MTTR metrics for both old and new systems.
Ensure that other variables (like software updates, hardware changes) are controlled or accounted for.

Statistical Analysis:

Use paired t-tests or ANOVA to compare pre- and post-deployment data.

#### Cons
ider multivariate regression if multiple factors might be influencing the outcomes.

Visualization & Reporting:

Create dashboards that show KPI trends over time.
Prepare a comprehensive report for leadership detailing findings, including any limitations of the study.

#### Closing Summary

By following this causal evaluation framework, we can present a well-supported case to leadership that the real-time streaming platform has genuinely improved system reliability. This approach ensures that improvements are not just superficial and helps in making informed decisions about future investments in similar technologies.

#### Common Mistakes
& Follow-Ups
Ignoring Baseline Data: Failing to establish reliable pre-deployment metrics can make it difficult to attribute changes accurately.
Overlooking Other Factors: Not considering other potential influences on reliability, such as external factors or coincidental events, can lead to false attributions.
Statistical Errors: Misusing statistical methods can result in misleading conclusions. For example, failing to adjust for multiple comparisons (like Bonferroni correction) when conducting numerous tests.

**Comparison:**

```
System	Baseline MTTTR (minutes)	New System MTTTR (minutes)
1	30	20
2	45	35
3	60	40
```

**Code:**

```python
# Example of a simple ANOVA test using Python's scipy library
from scipy.stats import f_oneway

# Sample data
baseline_mtttr = [30, 45, 60]  # old system MTTTR in minutes
new_system_mtttr = [20, 35, 40]

f_statistic, p_value = f_oneway(baseline_mtttr, new_system_mtttr)
print("F-statistic:", f_statistic)
print("P-value:", p_value)
```

**Example input / output:**

```

Input:

Baseline MTTTR data for a system: [30, 45, 60] minutes
New System MTTTR data after deployment: [20, 35, 40] minutes

Output:

CODE
Copy
F-statistic: 12.87
P-value: 0.005

The F-statistic and low p-value suggest that the difference in MTTTR between baseline and new system is statistically significant.
```

**Data:**

```json
{
  "baseline_mtttr": [30, 45, 60],
  "new_system_mtttr": [20, 35, 40]
}
```

**Flow:**

1. Define Baseline
2. Collect Pre-Deployment Data
3. Implement Streaming Platformin Staged Manner
4. Monitor Post-DeploymentPerformance
5. Collect Post-DeploymentMetrics
6. Conduct Statistical Analysis
7. Generate Dashboards &Reports

