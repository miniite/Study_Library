---
type: "practice"
difficulty: "Intermediate"
---

# dbt model contracts can enforce column types at build. How would you wire that so a breaking model change fails CI before it hits downstream BI?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Protobuf vs Avro Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Protobuf_vs_Avro_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that schema changes do not break downstream BI pipelines, we need to leverage dbt's model contracts feature effectively. Model contracts enforce column types at build time and can fail continuous integration (CI) if a breaking change is detected. This approach ensures data consistency and prevents issues in the downstream BI systems.

#### Triaging the Question

The question asks how to wire dbt model contracts so that any breaking changes in a model fail CI before reaching downstream BI pipelines. The key points are:

Understanding Model Contracts: dbt’s model contracts allow us to define constraints on our models, such as column types.
CI Integration: Ensuring that these contracts are checked during the CI process.
Breaking Changes: Detecting changes that could break downstream systems.

#### Assumptions
and Context
We have a working dbt project with multiple models and CI/CD pipelines in place.
The goal is to prevent column type mismatches or other breaking schema changes from propagating through our data pipeline.

#### First Principles Solution
Define Model Contracts: Use dbt's column directive within model contracts to define expected column types.
Integrate with CI/CD Pipeline: Ensure that the contract tests are run as part of the CI process.
Configure Alerts and Notifications: Set up alerts to notify relevant teams if a contract failure is detected.
Step-by-Step Implementation

Define Column Types in Model Contracts:

YML
# models/my_model.yml
version: 2
contracts:
  column_types:
    - name: customer_id
      type: int
      required: true
    - name: order_date
      type: date
      required: true

Integrate with CI/CD Pipeline: Ensure that the dbt docs generate command is included in your CI pipeline to run model contracts during the build process. Typically, this would be done using a .gitlab-ci.yml, GitHub Actions, or Jenkinsfile.

Configure Alerts and Notifications: Set up notifications via email or Slack if a contract failure is detected. This helps ensure that the issue is addressed promptly.

YAML
# .gitlab-ci.yml
stages:
  - test

dbt-verify-model-contracts:
  stage: test
  script:
    - dbt docs generate
  notifications:
    email:
      when: on_failure
      to: "data-team@example.com"
    slack:
      room: #channel-name
      message: ":red_circle: Model contracts failed in CI for project {{CI_PROJECT_NAME}}"

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By defining model contracts with column types and integrating these checks into the CI process, we can prevent breaking changes from impacting downstream BI systems. This ensures data consistency and reliable analytics.

#### Common Mistakes
Failing to define detailed contract rules, leading to insufficient validation.
Not configuring proper notifications, which means issues might go unnoticed until they affect production.
Ignoring the importance of type safety in schema changes.

#### Follow-Ups
How would you handle legacy data that doesn't match the new schema?
What about changes to non-column aspects like table names or relationships?
Can you describe a scenario where this approach might not be sufficient?

