---
type: "practice"
difficulty: "Intermediate"
---

# A hotfix pipeline skips the full test suite to go fast, but engineers started abusing it for normal changes; how do you technically constrain the fast path so it can only carry genuine hotfixes?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Emergency Deploy Paths](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Emergency_Deploy_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR:

To address the issue of a hotfix pipeline being misused, we need to implement technical constraints that ensure the fast path is only used for genuine hotfixes. This involves creating separate pipelines, customizing the CI/CD process, and establishing clear policies and communication practices.

#### Triaging the Question

The core question here is about ensuring that the "fast path" in a Continuous Integration/Continuous Deployment (CI/CD) pipeline is strictly reserved for true hotfixes while normal changes must go through a full suite of tests. The interviewer is probing our understanding of CI/CD best practices, technical constraints, and how to enforce policies within an organization.

#### Assumptions
Pipelines: There are multiple pipelines in place, including a main pipeline for regular development work and a hotfix pipeline.
Tests: Regular changes go through a full suite of tests (unit, integration, end-to-end), while hotfixes may have less stringent testing requirements but still need to be validated.
Team Practices: Engineers are aware that the fast path is reserved for hotfixes and understand the importance of adhering to this policy.

#### First-Principles Solution
1. Separate Pipelines

We can create separate pipelines for hotfixes and regular development changes. This ensures clear separation in the CI/CD process:

Regular Pipeline
Full Test Suite
Hotfix Pipeline
Gated Hotfix Tests
2. Customized Testing in Fast Path

For the hotfix pipeline, we can implement custom testing that is faster and less comprehensive than a full suite of tests but still sufficient to catch major issues.

Hotfix Pipeline
Gated Hotfix Tests
3. Automated Gate Checks

Implement automated gate checks to ensure that only changes that pass these gates can enter the fast path:

BASH
git push --set-upstream hotfixes origin/branch_name
if (gate_check_passed) {
    proceed_to_fast_path;
} else {
    reject_change;
}
4. Policy Enforcement

Establish clear policies and communication practices to remind engineers about the appropriate use of each pipeline.

#### Closing Summary

By implementing separate pipelines, customizing testing for hotfixes, using automated gate checks, and enforcing policies, we can ensure that the fast path is only used for genuine hotfixes while maintaining quality standards for regular changes.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to create clear separation between pipelines, leading to confusion.
Mistake 2: Over-relying on automation and not communicating the policy clearly to engineers.

#### Follow-Up
Question: How do you handle situations where developers bypass the automated gate checks?

#### Follow-Up
Question: What happens if a false positive occurs in the hotfix pipeline?

#### Example
of Automating Gate Checks:
BASH
#!/bin/bash

# Function to check if commit is a hotfix
is_hotfix() {
    git log -1 --pretty=%B | grep "hotfix" > /dev/null
}

# Function to run fast checks
run_fast_checks() {
    # Custom fast tests here
    echo "Running fast checks..."
    python3 ./scripts/fast_tests.py 2>&1 | tee results.txt
}

if is_hotfix; then
    run_fast_checks
else
    echo "This change does not qualify for the hotfix pipeline."
fi

# Check if tests passed
if grep -q "tests passed" results.txt; then
    git push --set-upstream hotfixes origin/branch_name
else
    echo "Tests failed. Change rejected."
fi

**Comparison:**

```
Pipeline	Description
Regular	Full test suite, comprehensive checks for all changes.
Hotfix	Custom fast tests, less stringent but still sufficient.
```

**Code:**

```bash
#!/bin/bash

# Function to check if commit is a hotfix
is_hotfix() {
    git log -1 --pretty=%B | grep "hotfix" > /dev/null
}

# Function to run fast checks
run_fast_checks() {
    # Custom fast tests here
    echo "Running fast checks..."
    python3 ./scripts/fast_tests.py 2>&1 | tee results.txt
}

if is_hotfix; then
    run_fast_checks
else
    echo "This change does not qualify for the hotfix pipeline."
fi

# Check if tests passed
if grep -q "tests passed" results.txt; then
    git push --set-upstream hotfixes origin/branch_name
else
    echo "Tests failed. Change rejected."
fi
```

**Example input / output:**

```text
$ git push --set-upstream hotfixes origin/branch_name

Running fast checks...
Tests passed.
remote: Analyzing commit message...
remote: This change qualifies as a hotfix.
To origin
   0123456..789abcdef  hotfixes -> hotfixes
```

**Data:**

```json
{
  "gate_check_passed": true,
  "commit_message": "hotfix: Fix critical bug in login system"
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Is this a hotfix?
7. Run custom fast checks
8. Tests passed?
9. Publish to hotfix pipeline
10. Raise error: Not a hotfix
11. Reject change

