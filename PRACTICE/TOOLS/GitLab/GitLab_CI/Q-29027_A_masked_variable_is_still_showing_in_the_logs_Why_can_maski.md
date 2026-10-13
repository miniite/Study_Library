---
type: "practice"
difficulty: "Deep"
---

# A masked variable is still showing in the logs. Why can masking fail and how do you make it robust?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Debugging Runner Failures](<../../../../TOOLS/GitLab/GitLab_CI/Debugging_Runner_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Masking variables in GitLab CI can fail due to several common issues, such as incorrect usage of the mask-variables setting or lack of proper configuration. To ensure robust masking, you need to correctly configure your GitLab pipeline and use environment-specific variable scopes.

#### Triaging the Question

The question is about understanding why a masked variable might still appear in logs despite being set to be masked. The core issue revolves around the mask-variables setting in .gitlab-ci.yml. The interviewer likely wants you to explain common pitfalls, how to properly configure masking, and provide best practices for ensuring variables are consistently masked.

#### First-Principles Solution
Understanding GitLab CI Variables: In GitLab CI/CD, variables can be used within the pipeline definitions. These variables are often sensitive data like API keys or passwords.
Masking Variables in .gitlab-ci.yml: To mask a variable, you use the mask-variables setting and specify which variables should be masked. However, there are several scenarios where masking might fail:
Incorrectly configured mask-variables.
Variables not being defined or referenced correctly.
Using incorrect variable scopes (e.g., global vs. job-specific).
Detailed Explanation

Masking Configuration:

YAML
mask-variables: [API_KEY, DB_PASSWORD]

This setting must be present and correctly configured within your .gitlab-ci.yml file.

Scope of Variables: Ensure that variables are defined at the correct scope (job-level or global). If a variable is job-specific but not masked in the mask-variables list, it won't be masked.

YAML
jobs:
  - name: example-job
    script:
      - echo "API_KEY=$API_KEY"
    variables:
      API_KEY: !mask 'your_secret_api_key'

Output Logs and Debugging:

Check the logs to see if the masked variable is still appearing.
Use the gitlab-runner debug mode or enable verbose logging to understand what’s happening during pipeline execution.

Environment-Specific Variables: Ensure that your CI/CD environment (e.g., local vs. production) uses different variables and scopes as necessary. This can help avoid accidental exposure of sensitive information.

YAML
stages:
  - test
  - deploy

test-job:
  stage: test
  script:
    - echo "API_KEY=$API_KEY"
  only:
    changes:
      - config/environments/test.rb

deploy-job:
  stage: deploy
  script:
    - echo "API_KEY=$API_KEY"
  environment:
    name: production
  variables:
    API_KEY: !mask 'your_production_api_key'

#### Common Mistakes
and Follow-Ups
Misplaced Masking: Forgetting to include certain variables in the mask-variables list.
Incorrect Variable Scopes: Not defining or referencing variables correctly within jobs.
Overwriting Masks: Using the same variable name across different scopes, leading to inconsistent masking.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, robust masking of sensitive variables in GitLab CI requires careful configuration. Ensure that all sensitive data is defined with the correct scope, included in mask-variables, and properly managed within your pipeline. Always debug logs and use verbose output to identify any issues.

Common pitfalls include:

Overlooking variables during the mask-variables list.
Incorrectly configuring variable scopes.
Failing to test masking thoroughly across different environments.

If you encounter persistent issues, consider reviewing the GitLab documentation or consulting with team members who have experience in this area.

