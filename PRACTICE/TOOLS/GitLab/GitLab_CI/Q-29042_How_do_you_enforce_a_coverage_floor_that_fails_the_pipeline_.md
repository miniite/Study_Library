---
type: "practice"
difficulty: "Intermediate"
---

# How do you enforce a coverage floor that fails the pipeline if coverage drops below the target on the default branch?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Debugging Runner Failures](<../../../../TOOLS/GitLab/GitLab_CI/Debugging_Runner_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a coverage floor in GitLab CI, we need to ensure that every commit or push to the default branch (usually main or master) is accompanied by a code coverage check that fails the pipeline if the coverage falls below a predefined threshold. This process involves setting up a custom job in your .gitlab-ci.yml file and configuring it with the necessary GitLab CI/CD features.

#### Triage
What’s Being Probed

The interviewer is likely interested in how you understand the concept of code coverage, the mechanics of GitLab CI pipelines, and your ability to implement conditional logic within a pipeline. They might also probe on best practices for testing and deployment, as well as potential pitfalls or edge cases.

#### Assumptions
You have basic knowledge of GitLab CI/CD.
The default branch is main.
A code coverage tool like lcov, pytest-cov, or similar is available in your project.
Your project supports the necessary command-line tools to generate and interpret code coverage reports.

#### First-Principles Solution
Step 1: Define Coverage Floor

First, determine the desired code coverage threshold. For example, let's use a minimum of 80% coverage on main.

Step 2: Add a Custom Job in .gitlab-ci.yml

You will need to add a custom job that runs before the main build or test steps and checks if the current coverage is below the desired threshold.

YAML
# .gitlab-ci.yml

stages:
  - test
  - quality

test_job:
  stage: test
  script:
    - echo "Running tests..."
    - make test  # Assuming a `make` command to run your tests and generate coverage reports

coverage_check_job:
  stage: quality
  when: on_success
  script:
    - coverage_report=$(lcov --summary coverage.info | grep TOTAL | awk '{print $4}')
    - if (( $(echo "$coverage_report < 80" | bc -l) )); then echo "Coverage is below the threshold"; exit 1; fi
Step 3: Handle Different Build Environments

If your project uses different build environments, ensure that the coverage tool and report generation process are consistent across them.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, by adding a custom job to check code coverage before allowing the pipeline to proceed, you can enforce a strict quality standard on your main branch. This ensures that every commit or push is scrutinized for maintainability and robustness. Some common mistakes include not correctly interpreting coverage reports, forgetting to update the threshold in multiple places (e.g., .gitlab-ci.yml, test scripts), and not handling edge cases where coverage might be temporarily low due to unrelated changes.

#### Follow-Ups
How do you handle false positives or negatives in your coverage reports?
What if a critical change requires reducing coverage for a short time? How would you manage that scenario?

**Comparison:**

```
Environment	Coverage Tool	Example Command
Linux	lcov	lcov --capture --directory . -o coverage.info
macOS / Windows	codecov	codecov -f json -r <repo-name>
```

**Code:**

```yaml
stages:
  - test
  - quality

test_job:
  stage: test
  script:
    - echo "Running tests..."
    - make test  # Assuming a `make` command to run your tests and generate coverage reports

coverage_check_job:
  stage: quality
  when: on_success
  script:
    - coverage_report=$(lcov --summary coverage.info | grep TOTAL | awk '{print $4}')
    - if (( $(echo "$coverage_report < 80" | bc -l) )); then echo "Coverage is below the threshold"; exit 1; fi
```

**Example input / output:**

| Input                | Output                  |
|----------------------|-------------------------|
| `coverage_report = 65` | Coverage is below the threshold (Pipeline fails) |
| `coverage_report = 81` | Pipeline passes          |

**Data:**

```json
{
  "coverage.info": {
    "TOTAL": {
      "lines_covered": 235,
      "total_lines": 340,
      "branch_coverage_percent": "78%",
      "line_coverage_percent": "69%"
    }
  }
}
```

**Flow:**

1. Yes
2. Coverage < Threshold
3. Coverage >= Threshold
4. Start
5. Is Coverage Check JobEnabled?
6. Run Tests
7. Generate Coverage Report
8. Check Coverage AgainstThreshold
9. Pipeline Fails
10. Pipeline Passes

