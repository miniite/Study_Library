---
type: "practice"
difficulty: "Expert"
---

# You must set a company standard for how database schema changes flow through CI/CD across hundreds of services and dozens of database engines; design the policy, the automation, and the guardrails that make expand-and-contract the default and destructive changes hard to do by accident.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Evolution in Event Driven Deploys](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Schema_Evolution_in_Event_Driven_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this role as a Backend Architect/Principal, we need to establish robust CI/CD practices for managing database schema changes across hundreds of services and dozens of database engines. Our goal is to make expand-and-contract operations easy while making destructive changes harder to execute accidentally. This requires defining clear policies, automating processes, and implementing guardrails.

#### Triage
What the Question is Really Asking

The question is asking us to design a comprehensive CI/CD strategy for database schema changes that balances ease of use with safety. Specifically, it aims to ensure:

Policy Design: Define rules around how database changes should be made.
Automation: Create tools and scripts to automate these changes.
Guardrails: Implement checks and balances to prevent accidental destructive changes.

#### Assumptions
We have multiple services that interact with databases.
Databases are managed using various engines (MySQL, PostgreSQL, etc.).
We need to handle schema changes efficiently while maintaining data integrity.
There is a team responsible for deploying these changes.
Step-by-Step Solution from First Principles
1. Define the Policy

The policy should be clear and concise:

Non-Destructive Changes (Expand-and-Contract): These are changes that add or modify fields without removing existing ones.
Destructive Changes: These include drops, renames, or other destructive actions.
Non-Destructive Changes
Schema migrations can be done using tools like Flyway or Liquibase.
Each service must use these tools to version its schema changes.
Every change is recorded and versioned in a changelog.
Destructive Changes
These should require manual approval before execution.
They may include database backups, detailed rollback plans, and notification of affected services.
2. Automation Setup

Automation is key to ensuring consistency and reducing errors:

Continuous Integration (CI): Use CI tools like Jenkins or GitLab CI/CD to run tests before deploying schema changes.
Database Migrations: Integrate database migration tools into the CI pipeline.
Notification System: Implement a notification system that alerts relevant stakeholders when destructive changes are requested.
3. Guardrails and Checks

Guardrails ensure that destructive changes do not occur by accident:

Manual Approval Workflows: Use pull requests or approval queues to review destructive changes before they are applied.
Automated Tests: Run schema tests (e.g., data integrity checks) as part of the CI pipeline.
Backup Before Changes: Ensure there is always a backup available for rollback in case of issues.
4. Implementation

Here’s how we can implement these concepts:

#### Example
Policy Document
MARKDOWN
### Database Schema Change Policy

1. All schema changes must be versioned and documented.
2. Non-destructive changes should use Flyway or Liquibase scripts.
3. Destructive changes require manual approval, database backups, rollback plans, and notifications to affected services.
4. Automated tests for data integrity must pass before any change is deployed.

### Example Pull Request Process
1. Developer opens a pull request with the schema change script.
2. CI runs automated tests.
3. If tests pass, it is reviewed by a senior engineer.
4. Destructive changes require additional approval steps.
CI/CD Pipeline Example (Simplified)
YAML
stages:
  - test
  - deploy

test_job:
  stage: test
  script:
    - ./run_tests.sh
  only:
    - master

deploy_job:
  stage: deploy
  script:
    - flyway_migrate.sh
  when: manual

#### Closing Summary
+ Common Mistakes/Follow-Ups

By setting up a clear policy, automating the process, and implementing robust guardrails, we can ensure that expand-and-contract operations are easy while making destructive changes harder to execute by accident. Key takeaways include:

Automation: Automate as much of the process as possible to reduce errors.
Guardrails: Implement strong checks and balances to prevent accidental destructive changes.
Documentation: Ensure all processes are well-documented for clarity.

#### Common mistakes
might include neglecting to implement automated tests, failing to require manual approval for critical changes, or not providing sufficient rollback plans. Follow-up questions could involve discussing specific technologies or tools used in the pipeline, integration with existing infrastructure, and how to handle legacy systems that do not support modern CI/CD practices.

**Comparison:**

| Stage | Script | Action |
|-------|--------|--------|
| Test  | run_tests.sh | Runs automated tests |
| Deploy | flyway_migrate.sh | Deploys schema changes (requires manual approval for destructive) |

**Code:**

```yaml
stages:
  - test
  - deploy

test_job:
  stage: test
  script:
    - ./run_tests.sh
  only:
    - master

deploy_job:
  stage: deploy
  script:
    - flyway_migrate.sh
  when: manual
```

**Example input / output:**

```text
Input:
- A pull request with a schema change script using Flyway.
Output:
1. CI runs tests and approves the non-destructive changes automatically.
2. For destructive changes, manual approval is required before deployment.
3. If automated tests fail, changes are not deployed.
```

**Data:**

```json
{
  "version": "1.0",
  "changelog": {
    "2023-09-01": [
      { "type": "non-destructive", "description": "Added new field for user preferences" },
      { "type": "destructive", "description": "Removed obsolete logging table" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Developer Opens PR
6. CI Runs Tests
7. (Tests Pass)
8. Automated DeployNon-Destructive Changes
9. (Tests Fail)
10. Notify Developer to Fix Errors
11. (PR Closed)
12. Manual Approval forDestructive Changes
13. (Approval Given)
14. Deploy Changes
15. Manual Rollback Plan
16. Deploy Rejected
17. CI Pipeline Ends

