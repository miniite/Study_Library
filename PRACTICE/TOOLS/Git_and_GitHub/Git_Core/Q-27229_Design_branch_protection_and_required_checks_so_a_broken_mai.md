---
type: "practice"
difficulty: "Deep"
---

# Design branch protection and required checks so a broken main is nearly impossible for the training repo (main must always be trainable), while still allowing an audited emergency break-glass override. How would you record and review those overrides?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Branch Protection Design](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branch_Protection_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the main branch of a training repository is nearly impossible to break, we need to design robust branch protection rules and enforce required checks. This will involve implementing several safeguards, such as requiring pull requests (PRs) with multiple approvals, automated code checks, and mandatory tests before merging into main. Additionally, an emergency "break-glass" override process must be in place to handle unforeseen situations while maintaining auditability.

#### Triaging the Question

The key points this question is probing are:

Branch Protection Rules: How we enforce rules on branches to prevent accidental or malicious breaks.
Automated Checks: Ensuring that all changes undergo rigorous automated testing before they can be merged.
Emergency Override Mechanism: A process for quickly resolving critical issues without breaking the usual safety net, and how this mechanism is audited.

#### Assumptions
The repository has multiple contributors with different roles (e.g., developers, reviewers).
There are existing workflows and tools for code reviews and testing.
Emergency overrides need to be rare but must not compromise security or reliability.

#### First-Principles Solution
1. Branch Protection Rules
Main Branch: Enforce strict rules on the main branch.
Only allow merging from protected branches (e.g., staging, feature-branches).
Require at least one approving reviewer and a required status check to pass before merging.
staging
main
Main
Staging
Feature-Branches
2. Automated Checks
Implement mandatory checks such as unit tests, integration tests, and code quality metrics.
Use CI/CD pipelines to run these checks automatically before any merge.
CI/CD
Run Unit Tests
Run Integration Tests
Code Quality Check
3. Emergency Override Mechanism
Create a break-glass branch specifically for emergency overrides.
Allow manual merges into the main branch from this branch only if certain criteria are met (e.g., signed off by multiple senior stakeholders).
Maintain an audit log of all such overrides to track who did what and when.
Emergency Override
Manual Merge into Main
Audit Log
Recording and Reviewing Overrides

To record and review these emergency overrides:

Override Request: When an override is needed, create a pull request from the break-glass branch to the main branch.
Review Process: The override PR must be reviewed by at least two senior stakeholders who understand the urgency of the situation.
Approval and Merge: Only after approval can the override be merged manually into the main branch.
Audit Logging: Log all details about the override in a central audit log, including:
Who initiated the request
Why it was necessary
What changes were made
When it occurred

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, we need to design a system that ensures main remains robust while allowing rare emergencies to be handled. The key components are strict branch protection rules, automated checks, and an emergency override mechanism with detailed audit logging.

#### Common Mistakes

Failing to enforce strong branch protection on the main branch.
Not having a clear process for emergency overrides that includes multiple approvals.
Ignoring the need for thorough documentation and audit trails of these overrides.

#### Follow-Ups

#### Cons
ider integrating a feedback loop where post-override reviews are conducted to identify any weaknesses or improvements.
Ensure regular training for stakeholders on the override process to maintain consistency.

**Comparison:**

| Initiator | Reason                 | Timestamp         |
|-----------|------------------------|-------------------|
| John Doe  | Urgent fix for production issue | 2023-10-05T14:00:00Z |

**Code:**

```python
# Example Python code for automated checks in CI/CD pipeline

def run_unit_tests():
    # Run unit tests and return results
    pass

def run_integration_tests():
    # Run integration tests and return results
    pass

def check_code_quality():
    # Check code quality metrics and return results
    pass

def main():
    if all([run_unit_tests(), run_integration_tests(), check_code_quality()]):
        print("All checks passed. Proceed with merge.")
    else:
        print("Checks failed. Merge blocked.")

if __name__ == "__main__":
    main()
```

**Example input / output:**

```text
Input:
Override request from break-glass branch to main branch

Output:
Audit log entry created with details of the override.
Manual merge into main branch approved by senior stakeholders.

Dry-run example:
Emergency Override Request initiated by John Doe for "Urgent fix for production issue".
Approval process initiated. Approvals received from Sarah Smith and Emily Brown.
Override merged manually into main branch at 2023-10-05T14:00:00Z.
Audit log updated with all details of the override.
```

**Data:**

```json
{
  "branch_protection": {
    "rules": [
      {
        "branch": "staging",
        "required_checks": ["unit-tests", "integration-tests"]
      },
      {
        "branch": "main",
        "protected_branches": ["staging"],
        "required_reviews": 2
      }
    ]
  },
  "emergency_override": {
    "break_glass_branch": "break-glass-override",
    "approvals_required": 3,
    "audit_log_entry": [
      {
        "initiator": "John Doe",
        "reason": "Urgent fix for production issue",
        "timestamp": "2023-10-05T14:00:00Z"
      }
    ]
  }
}
```

**Flow:**

1. Initiate Override Request
2. Create PR from break-glassbranch
3. Review Process (at least 2approvals)
4. Manual Merge into mainbranch if approved
5. Audit Log Entry Created

