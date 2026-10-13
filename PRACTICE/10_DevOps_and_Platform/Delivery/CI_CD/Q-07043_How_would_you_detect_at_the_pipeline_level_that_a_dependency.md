---
type: "practice"
difficulty: "Deep"
---

# How would you detect at the pipeline level that a dependency upgrade changed behavior in a way tests don't cover (e.g., a JSON serialization default flipped), before it causes subtle prod bugs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Infrastructure Drift Detection](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Infrastructure_Drift_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to ensure that our CI/CD pipeline can detect changes in behavior introduced by dependency upgrades, particularly those not caught by existing tests. This involves a multi-layered approach focusing on code analysis, performance monitoring, and behavioral testing.

Firstly, let's clarify what the question is probing:

Code Analysis: We need tools to analyze source code for potential changes.
Behavioral Testing: Implementing or enhancing functional tests that simulate real-world scenarios more closely than traditional unit tests.
Performance Monitoring: Setting up monitoring systems to catch unexpected behavior.

#### Assumptions
and key points include:

Dependency upgrades are common in software development.
Traditional testing might not cover all edge cases, especially those related to data handling like JSON serialization defaults.
We need a comprehensive approach that goes beyond basic unit tests to catch subtle changes.
Step-by-Step Solution

Dependency Version Management:

Use tools like pip, npm, or Maven to manage and track dependency versions in your project.
Implement automated checks during the build process to verify that only approved versions of dependencies are used.

Behavioral Testing Frameworks:

Integrate behavioral testing frameworks like JBehave, Cucumber, or similar for Java/Python projects.
Write tests that simulate user interactions and system states, covering edge cases not covered by unit tests.

#### Example
For JSON serialization defaults, create test scenarios where different versions of the dependency handle null values differently.

Static Code Analysis:

Use static analysis tools like SonarQube, ESLint, or Pylint to detect potential issues in your codebase.
Set up rules and configurations to specifically look for changes that might impact serialization, such as default value modifications.

Runtime Behavior Monitoring:

Implement APM (Application Performance Management) tools like New Relic, Datadog, or Jaeger.
Monitor system performance metrics during the CI/CD pipeline runs, especially in stress tests and load testing scenarios.
Alert on any deviations from expected behavior patterns that might indicate a dependency change issue.

Manual Review and Documentation:

Have a process where critical changes are reviewed by experienced engineers to catch anything missed by automated tools.
Maintain detailed documentation of known issues and changes in dependencies, ensuring that all team members are aware.

Post-Deployment Monitoring:

Implement post-deployment monitoring with the same APM tools used during the CI/CD pipeline.
Set up canary releases to gradually roll out new versions and monitor user impact before full deployment.

#### Closing Summary

To detect changes in behavior introduced by dependency upgrades, a combination of code analysis, behavioral testing, static analysis, runtime monitoring, manual reviews, and post-deployment monitoring is essential. This approach ensures that even subtle issues like JSON serialization defaults are caught early, preventing them from causing production bugs.

#### Common Mistakes

Relying solely on unit tests.
Not having a robust CI/CD pipeline setup.
Ignoring static code analysis tools.
Failing to monitor runtime behavior and user interactions.

**Comparison:**

```
Component	Description
Static Analysis	Detects potential code changes using tools like ESLint or Pylint
Behavioral Tests	Simulates real-world scenarios to catch edge cases not covered by unit tests
APM	Monitors system performance during CI/CD and post-deployment
```

**Code:**

```python
# Example of a simple function to check JSON serialization defaults with an expected value
import json

def test_serialization_default():
    obj = {"key": None}
    expected_json = '{"key": null}'
    actual_json = json.dumps(obj, default=str)

    if actual_json != expected_json:
        raise ValueError("JSON serialization default changed unexpectedly")
```

**Example input / output:**

```text
| Input | Expected Output | Actual Output |
|-------|-----------------|---------------|
| `{"key": null}` | `{"key": null}` | `{"key": ""}` |

This dry run shows a potential issue with JSON serialization defaults.
```

**Data:**

```json
{
  "obj": {
    "key": null
  },
  "expected_json": "{\"key\": null}",
  "actual_json": "{\"key\": \"\"}"
}
```

**Flow:**

1. Trigger Pipeline
2. Code Analysis
3. Behavioral Testing
4. Runtme Monitoring
5. Manual Review
6. Post-Deployment Monitoring

