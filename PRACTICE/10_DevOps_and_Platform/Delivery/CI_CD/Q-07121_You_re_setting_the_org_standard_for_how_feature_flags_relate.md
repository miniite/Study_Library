---
type: "practice"
difficulty: "Expert"
---

# You're setting the org standard for how feature flags relate to deploys; teams are drowning in stale flags and inconsistent flag behavior across services. Design the flag-lifecycle governance and platform tooling that keeps flags fast, consistent, and reliably cleaned up at scale.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Feature Flags and Deploy Risk](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Feature_Flags_and_Deploy_Risk.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In this scenario, we need to design a robust feature flag lifecycle governance system that ensures consistency, reliability, and performance across multiple services. The key is to establish clear processes, automate critical tasks, and leverage modern CI/CD practices.

#### Triaging the Question

The question requires designing an end-to-end solution for managing feature flags in a multi-service architecture. This involves understanding current pain points (stale flags, inconsistent behavior), defining governance principles, and developing tooling that automates flag management to scale with growing team size and complexity.

#### Assumptions
Service Architecture: Multiple microservices, each potentially using different feature flags.
Team Size: More than 50 developers across multiple teams.
Flag Behavior Variability: Flags should behave consistently across services.
Scalability: The solution needs to handle thousands of flag changes per day without performance degradation.
Step-by-Step Solution:
1. Define Governance Principles

Purpose: Ensure all feature flags are managed consistently and securely.

Principle 1 (Visibility): All developers should see the latest state of feature flags.
Principle 2 (Consistency): Flags must behave identically across services.
Principle 3 (Security): Access to flags should be controlled and auditable.
2. Design a Centralized Flag Management Platform

This platform will serve as the single source of truth for all feature flags:

Database Schema: A database table for storing flag details, including flag_name, status, description, created_date, and last_modified.
API Endpoints:
GET /flags: Retrieve a list of all active flags.
POST /flags: Create or update a feature flag.
DELETE /flags/{id}: Delete a specific flag.
3. Integrate with CI/CD Pipelines

Ensure that changes to feature flags trigger deployments:

CI Pipeline Step: A step in the CI pipeline that checks for new or updated flags before merging them into production branches.
Triggering Deploys: Use webhooks to notify deployment pipelines when a flag is modified.
4. Implement Flag Validation and Auditing

Ensure flags are used correctly across services:

Flag Validation Plugin: A plugin in each service that checks if the correct conditions are met before enabling new features.
Audit Logs: Maintain logs of all flag changes for audit purposes.
Sample Data Schema (JSON)
JSON
{
  "flags": [
    {
      "flag_name": "NEW_USER_TRIAL",
      "status": "active",
      "description": "Enables the trial mode for new users.",
      "created_date": "2023-10-01T00:00:00Z",
      "last_modified": "2023-10-05T14:30:00Z"
    },
    {
      "flag_name": "PREMIUM_UPSELL",
      "status": "inactive",
      "description": "Upsells users to premium plans.",
      "created_date": "2023-10-02T10:00:00Z",
      "last_modified": null
    }
  ]
}

#### Example
CI Pipeline Step (YAML)
YAML
steps:
  - name: Check for Flag Changes
    image: alpine:latest
    commands:
      - curl --silent https://api.flag-manager.example.com/flags | jq '.flags[].flag_name' > flag_changes.json
      - if [ "$(cat flag_changes.json)" != "" ]; then echo "New flags detected. Proceeding with deploy..."; fi
Worked Input-Output Example (Dry Run)
PLAINTEXT
Input:
  - New feature flag request: NEW_USER_TRIAL, active status.

Output:
  - Flag created in the database.
  - CI pipeline triggered to validate new flag usage across services.
  - Deployment pipelines notified of changes and proceed with deployment.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The proposed solution ensures consistency, reliability, and automation for feature flags. It leverages a centralized management platform integrated into the CI/CD process, providing visibility and security.

#### Common Mistakes

Failing to integrate flag changes into CI/CD pipelines.
Not implementing validation checks in services.
Underestimating the need for audit logs and visibility tools.

#### Follow-Ups

How would you handle edge cases like emergency flags that bypass regular approval processes?
What measures would you take to ensure flags are cleaned up when no longer needed?

**Comparison:**

```
Feature	Description
Visibility	All developers see the latest state of feature flags.
Consistency	Flags behave identically across all services.
Security	Access to flags is controlled and auditable.
```

**Code:**

```yaml
# CI Pipeline Step Example in YAML
steps:
  - name: Check for Flag Changes
    image: alpine:latest
    commands:
      - curl --silent https://api.flag-manager.example.com/flags | jq '.flags[].flag_name' > flag_changes.json
      - if [ "$(cat flag_changes.json)" != "" ]; then echo "New flags detected. Proceeding with deploy..."; fi
```

**Example input / output:**

```text
Input:
- New feature flag request: NEW_USER_TRIAL, active status.

Output:
- Flag created in the database.
- CI pipeline triggered to validate new flag usage across services.
- Deployment pipelines notified of changes and proceed with deployment.
```

**Data:**

```json
{
  "flags": [
    {
      "flag_name": "NEW_USER_TRIAL",
      "status": "active",
      "description": "Enables the trial mode for new users.",
      "created_date": "2023-10-01T00:00:00Z",
      "last_modified": "2023-10-05T14:30:00Z"
    },
    {
      "flag_name": "PREMIUM_UPSELL",
      "status": "inactive",
      "description": "Upsells users to premium plans.",
      "created_date": "2023-10-02T10:00:00Z",
      "last_modified": null
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Feature Flag Request
4. Is the flag new?
5. Create/Update in DB
6. Check if CI Pipeline StepTriggers
7. Trigger CI Pipeline Step
8. Notify Deployment Pipelinesof Changes
9. CI Validates Flag Usage AcrossServices
10. Deploy New Flags

