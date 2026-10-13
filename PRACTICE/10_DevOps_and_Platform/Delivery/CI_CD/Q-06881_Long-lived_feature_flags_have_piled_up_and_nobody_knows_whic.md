---
type: "practice"
difficulty: "Deep"
---

# Long-lived feature flags have piled up and nobody knows which are safe to remove; how do you build a lifecycle and pipeline check that forces flag cleanup?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Feature Flags and Deploy Risk](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Feature_Flags_and_Deploy_Risk.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a CI/CD pipeline and lifecycle management system to help identify and safely remove long-lived feature flags. The goal is not just automation but ensuring that the process is safe and systematic, reducing risk and improving codebase hygiene.

#### Triage

The question is about building an automated pipeline to manage and clean up feature flags in a software project. It's probing how you would design a robust system that can identify unused or unnecessary feature flags and ensure they are removed safely without disrupting the application. The interviewer might be interested in your approach to balancing automation with safety, as well as considering potential trade-offs between different solutions.

#### First Principles Solution
Identify Feature Flags: We need a way to list all active feature flags.
Assess Usage: Determine if any of these flags are still being used by checking their status in the codebase and usage logs.
Risk Assessment: Evaluate which flags can be safely removed based on historical data, feedback from stakeholders, and risk assessment.
Automation with Safety Nets: Design a CI/CD pipeline that enforces safe flag removal, ensuring no critical functionality is impacted.
Step-by-Step Solution:

Feature Flag Inventory:

Use a database or a configuration management tool to store information about feature flags (e.g., name, description, status).

Usage Detection:

Integrate with code scanning tools like SonarQube or static analysis tools such as Pylint or ESLint.
Implement logging and monitoring in the application to track which parts of the system are using each flag.

Risk Assessment:

Use historical data from code changes, deployment logs, and issue tracking systems (e.g., JIRA) to assess the impact of removing a flag.
Engage with stakeholders and operational teams for feedback on critical flags that should not be removed.

CI/CD Pipeline Design:

Create a pipeline that periodically checks the feature flag inventory against usage data and risk assessments.
When a flag is identified as safe to remove, create a removal task in the CI/CD pipeline with appropriate approvals.
Implement dry-run capabilities so changes can be tested without impacting production.

#### Implementation

Use tools like Jenkins, GitHub Actions, or GitLab CI for building and deploying the pipeline.
Ensure that each step is well-documented and auditable to maintain transparency and trust in the process.

Monitoring and Feedback Loop:

Set up alerts for any issues detected during the removal process.
Continuously collect feedback from users and stakeholders to refine the process over time.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
A common mistake is to rush into flag removal without proper assessment, which can lead to unintended service disruptions or user experience degradation.
Another pitfall is not involving all relevant parties in the decision-making process, leading to oversight of critical use cases.
It's important to have a robust monitoring system to catch any issues early and ensure that the removal process does not introduce new problems.

#### Follow-Up
Questions:
How would you handle flags that are being used by third-party integrations or plugins?
What measures can be taken to prevent accidental removal of critical feature flags during automated processes?

**Comparison:**

```
Flag Name	Status	Last Used Date
flagA	active	2023-10-05
flagB	inactive	NULL
flagC	active	2023-09-15
Static Analysis Results
flagA: present in code
flagB: not found
flagC: present in code
```

**Code:**

```python
# Example code snippet for flag inventory check in a CI/CD pipeline

import requests

def fetch_feature_flags():
    response = requests.get('https://api.featureflagstore.com/v1/flags')
    return response.json()

def check_usage(usage_logs):
    used_flags = set()
    for log in usage_logs:
        if 'feature_flag' in log['event']:
            used_flags.add(log['event']['feature_flag'])
    return used_flags

# Example integration with a static analysis tool
static_analysis_tool = 'sonarqube'
analysis_results = fetch_static_analysis_data(static_analysis_tool)
used_features = check_usage(analysis_results)
```

**Example input / output:**

```text
Input:
- Feature flags from API call: [flagA, flagB, flagC]
- Usage logs: [flagA]

Output:
- Flags to be removed: [flagB, flagC] (assuming no approval process is in place)
```

**Data:**

```json
{
  "feature_flags": [
    {"name": "flagA", "status": "active"},
    {"name": "flagB", "status": "inactive"},
    {"name": "flagC", "status": "active"}
  ],
  "usage_logs": [
    {"event": {"feature_flag": "flagA"}},
    {"event": {"feature_flag": "flagD"}}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Are feature flags active?
5. Fetch usage data and assessrisk
6. Identify safe to remove flags
7. Create CI/CD pipeline tasks forremoval
8. Monitor and feedback
9. Check with stakeholders
10. Verify critical use cases

