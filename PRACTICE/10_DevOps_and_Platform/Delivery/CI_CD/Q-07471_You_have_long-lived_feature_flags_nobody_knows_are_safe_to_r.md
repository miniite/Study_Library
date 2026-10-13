---
type: "practice"
difficulty: "Intermediate"
---

# You have long-lived feature flags nobody knows are safe to remove; how would you build a lifecycle and a pipeline check that forces flag cleanup without breaking anything still depending on them?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Feature Flags and Deploy Risk](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Feature_Flags_and_Deploy_Risk.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of long-lived feature flags that might no longer be needed, we need to build a robust lifecycle management system for these flags within our CI/CD pipeline. This involves identifying unused or redundant flags and ensuring that any changes in the codebase do not break dependent components. Here’s a step-by-step approach:

#### Triage
What We Are Addressing

The core question revolves around developing a mechanism to systematically identify feature flags that are safe for removal while maintaining the integrity of the application. The primary goal is to avoid breaking existing dependencies by ensuring that critical features remain operational until they can be safely removed.

#### Assumptions
and Key Considerations
Codebase Complexity: Our application has multiple components, services, and databases interconnected.
Feature Flags Usage: Feature flags are frequently used for A/B testing, feature toggles, or fallback mechanisms.
CI/CD Pipeline: We have a mature CI/CD pipeline with various stages such as build, test, deploy.

#### First-Principles Solution

Dependency Analysis:

Use static code analysis tools to identify all features and dependencies within the application.
Analyze logs and metrics from monitoring systems to find usage patterns of these feature flags.

Automated Tests:

Develop unit tests and integration tests that cover different scenarios where these feature flags are used.
Ensure that any changes do not break existing functionality by running a suite of automated tests.

Impact Assessment:

Implement a risk assessment framework to evaluate the potential impact of removing each flag.
Use a scoring system based on factors like code complexity, usage frequency, and business-criticality.

Flag Deprecation Policy:

Create a policy that mandates at least 30 days from initial identification before flags can be removed.
Ensure all stakeholders are notified and given time to adapt or refactor their dependencies.

CI/CD Pipeline Integration:

Extend the CI pipeline to include flag lifecycle management as part of its stages.
Use pre-deployment checks to validate that dependent components still function correctly after a flag is removed.

#### Implementation
Steps

Static Code Analysis:

BASH
# Example command for SonarQube analysis
sonar-scanner

Log and Metric Analysis:

Query logs and metrics using tools like Prometheus, Grafana.

#### Example
query in Prometheus:
CODE
count without(feature_flag) (increase(our_service_request_total{feature_flag!="none"}[1d]))

Automated Tests:

PYTHON
import unittest

class TestFeatureFlag(unittest.TestCase):
    def test_feature_flag_usage(self):
        # Simulate feature flag usage and assert that it works as expected
        pass

Risk Assessment Framework:

Define scoring criteria (e.g., 10 points for critical features, 5 for non-critical).
Implement a script to calculate scores:
PYTHON
def assess_risk(flag_usage):
    score = 0
    if flag_usage['critical']:
        score += 10
    if flag_usage['usage_frequency'] > 1000:
        score -= 5
    return score

CI/CD Pipeline Integration:

Add a new stage to the pipeline for flag deprecation.

#### Example
pipeline configuration in Jenkinsfile:
GROOVY
stage('Flag Deprecation') {
    steps {
        script {
            // Run static analysis, tests, and risk assessment
            def results = analyze()
            if (results.impact > 5) {
                error("High impact flag found")
            }
            remove_flag(results.flag)
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

By systematically identifying and managing unused feature flags, we can ensure that our application remains robust and maintainable. This approach involves a combination of static analysis, automated testing, risk assessment, and integration into the CI/CD pipeline.

#### Common mistakes
to avoid include:

Failing to notify stakeholders about upcoming changes.
Removing flags too quickly without adequate validation.
Not integrating flag deprecation checks into regular deployment cycles.

#### Follow-up
questions might explore how to handle edge cases, such as flags used in third-party integrations or legacy systems. Additionally, discussions could extend to optimizing resource usage by retiring unused features more effectively.

**Comparison:**

| Feature | Criticality | Usage Frequency | Score |
|---------|-------------|----------------|-------|
| A       | False       | 10             | -5    |
| B       | True        | 5000           | -3    |

**Code:**

```

sonar-scanner

CODE
Copy

```python
import unittest

class TestFeatureFlag(unittest.TestCase):
    def test_feature_flag_usage(self):
        # Simulate feature flag usage and assert that it works as expected
        pass

def assess_risk(flag_usage):
    score = 0
    if flag_usage['critical']:
        score += 10
    if flag_usage['usage_frequency'] > 1000:
        score -= 5
    return score

def remove_flag(results.flag)
GROOVY
Copy
stage('Flag Deprecation') {
    steps {
        script {
            // Run static analysis, tests, and risk assessment
            def results = analyze()
            if (results.impact > 5) {
                error("High impact flag found")
            }
            remove_flag(results.flag)
        }
    }
}
```

**Example input / output:**

```text
# Example input for static code analysis
$ sonar-scanner

# Example output from risk assessment script
Flag 'FEATURE_B' has a score of -3 and is flagged for removal.

# Example output from flag deprecation pipeline stage
Pipeline failed at Flag Deprecation stage due to high impact on FEATURE_C.
```

**Data:**

```json
{
  "feature_usage": [
    {"flag_name": "FEATURE_A", "critical": false, "usage_frequency": 10},
    {"flag_name": "FEATURE_B", "critical": true, "usage_frequency": 5000}
  ]
}
```

**Flow:**

1. Identify unused flags
2. Static Code Analysis
3. Automated Tests
4. Risk Assessment
5. Deprecation Policy
6. CI/CD Pipeline Integration

