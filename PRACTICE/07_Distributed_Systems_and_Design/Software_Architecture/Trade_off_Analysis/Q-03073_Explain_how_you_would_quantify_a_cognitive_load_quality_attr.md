---
type: "practice"
difficulty: "Expert"
---

# Explain how you would quantify a 'cognitive load' quality attribute for a full-stack service so an ATAM can analyze it — e.g. 'a new engineer ships a safe change to the checkout flow within their first week.' What makes this measurable rather than hand-wavy?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To quantify cognitive load in a full-stack service, we need to define and measure it in terms of specific, quantifiable metrics that can be analyzed as part of an Architectural Trade-off Analysis (ATAM). Cognitive load refers to the mental effort required from developers or operators when interacting with the system. For instance, if a new engineer can safely ship changes to the checkout flow within their first week, we need to measure this cognitive load and ensure it is manageable.

Let's break down how we would approach quantifying cognitive load for a full-stack service:

Define Cognitive Load Metrics: We need to identify key metrics that represent the mental effort required by developers or operators.
Collect Data: Gather data from various sources such as code reviews, deployment logs, and developer surveys.
Analyze Patterns: Use statistical analysis and visualization tools to understand patterns in cognitive load over time.
Compare Scenarios: Compare different development scenarios to identify the factors that influence cognitive load.

To make this measurable rather than hand-wavy, we will use a combination of empirical data collection and quantitative metrics.

#### Triage
What's Being Probed

The core of the question is how to define and measure cognitive load in such a way that it can be systematically analyzed as part of an ATAM. The interviewer is likely probing:

Empirical Metrics: Can you identify specific, measurable indicators of cognitive load?
Data Collection Methods: How would you collect data on these metrics?
Analytical Techniques: What tools and methods would you use to analyze the collected data?

#### Assumptions
The full-stack service has a well-defined development and deployment process.
There is existing infrastructure for logging and monitoring (e.g., GitLab, JIRA).
Developers and operators are willing to participate in surveys or provide feedback.
Step-by-Step Solution
1. Define Cognitive Load Metrics

Cognitive load can be broken down into several components:

Complexity of Code: Measured by lines of code (LOC), cyclomatic complexity, and technical debt.
Integration Points: Number of inter-service dependencies or external APIs.
Error Handling: Frequency and severity of errors reported during development and deployment.
Documentation Quality: Accessibility and completeness of documentation.
2. Collect Data

We can collect data from various sources:

Code Reviews: Count the number of issues raised per pull request.
Deployment Logs: Track deployment failures, rollback rates, and mean time to recovery (MTTR).
Surveys/Feedback: Conduct regular surveys among developers to understand their perception of cognitive load.
3. Analyze Patterns

Use statistical methods to analyze the collected data:

Descriptive Statistics: Calculate averages, medians, and standard deviations.
Correlation Analysis: Identify relationships between cognitive load metrics and deployment outcomes.
Trend Analysis: Visualize trends over time using line charts or heatmaps.
4. Compare Scenarios

Compare different development scenarios to understand the impact of changes:

Baseline vs. Experimental: Compare a baseline scenario with an experimental change (e.g., introducing new features, refactoring code).
Before and After Metrics: Measure cognitive load before and after significant changes in the system.

#### Example Scenario

Scenario: New Engineer onboarding

Metrics to Track:
Lines of Code added per day.
Time spent on code reviews (in hours).
Number of issues found during deployments.
Feedback from surveys on perceived cognitive load.

Data Collection:

Developers are surveyed at the end of their first week, asking how much time they spent debugging, understanding existing code, and documenting new features.
Code review logs show the number of issues raised per pull request by a new engineer in their first week.
Deployment logs track any rollback attempts or errors encountered during deployment.

Analysis:

Calculate average lines of code added, average review time, frequency of issues found, and perceived cognitive load score from surveys.
Compare these metrics with those from previous onboarding cycles to identify trends.

#### Common Mistakes/Follow-Ups
Over-reliance on Metrics: Focusing too much on quantitative data without considering qualitative feedback from developers.
Ignoring Developer Feedback: Relying solely on data-driven insights without incorporating developer sentiment and subjective experiences.
Lack of Context: Not considering the context in which metrics are collected, such as team size or project complexity.

By systematically defining cognitive load metrics, collecting relevant data, analyzing patterns, and comparing scenarios, we can make cognitive load measurable and actionable for an ATAM.

**Comparison:**

| Day       | Lines of Code Added | Review Time (Hours) | Issues Found | Cognitive Load Score |
|-----------|--------------------|---------------------|--------------|----------------------|
| Before    | 20                 | 2                   | 1            | 2                    |
| Baseline  | 30                 | 4                   | 2            | 3                    |
| Experiment| 45                 | 6                   | 3            | 4                    |

**Code:**

```
MERMAID
Copy
flowchart TD
    subgraph DataCollection
        A[Code Reviews] --> B1(Calculate Issues)
        C[Deployment Logs] --> D1(Track Failures)
        E[Surveys/Feedback] --> F1(Score Cognitive Load)
    end

    subgraph Analysis
        G1[Descriptive Stats] --> H(Trend Analysis)
        I[Correlation Analysis] --> J(Identify Relationships)
    end

    subgraph Comparison
        K[Baseline vs. Experimental] --> L(Before and After Metrics)
    end

    A --> B1 --> C --> D1 --> E --> F1 --> G1 --> H
    I --> J --> K --> L
```

**Example input / output:**

| Day | Lines of Code Added | Review Time (Hours) | Issues Found | Cognitive Load Score |
|-----|-------------------|--------------------|--------------|----------------------|
| 0   | 25                | 3                  | 2            | 3                    |
| 1   | 30                | 5                  | 4            | 4                    |
| 2   | 40                | 7                  | 6            | 5                    |

**Data:**

```json
{
  "metrics": {
    "lines_of_code": {
      "added_per_day": [
        { "day": 0, "value": 25 },
        { "day": 1, "value": 30 },
        { "day": 2, "value": 40 }
      ]
    },
    "review_time": {
      "hours_spent": [
        { "day": 0, "value": 3 },
        { "day": 1, "value": 5 },
        { "day": 2, "value": 7 }
      ]
    },
    "issues_found": {
      "count": [
        { "day": 0, "value": 2 },
        { "day": 1, "value": 4 },
        { "day": 2, "value": 6 }
      ]
    },
    "cognitive_load_score": {
      "score": [
        { "day": 0, "value": 3 },
        { "day": 1, "value": 4 },
        { "day": 2, "value": 5 }
      ]
    }
  }
}
```

**Flow:**

1. Define Metrics
2. Collect Data
3. Aggregate Data
4. Analyze Trends
5. Compare Scenarios
6. Quantify Cognitive Load

