---
type: "practice"
difficulty: "Deep"
---

# You are deploying Helm charts from CI. How do you manage per-environment values and secret injection safely?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Environments](<../../../../TOOLS/GitLab/GitLab_CI/Protected_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To manage per-environment values and secrets safely in GitLab CI when deploying Helm charts, we need to ensure separation of concerns between our default values.yaml and environment-specific values. This involves using multiple stages, templating with variables, and leveraging GitLab's secret management capabilities.

#### Triaging the Question

The question is about how to manage configuration differences across environments (e.g., development, testing, production) while keeping secrets secure. The core concern is ensuring that sensitive data like database credentials or API keys are not exposed in source code repositories. We'll break down this problem from first principles and provide a comprehensive solution.

#### First-Principles Solution

Environment-Specific Values:

Create separate values-production.yaml, values-testing.yaml, etc., files for each environment.
Store these files under .helm/ or in the same repository directory, ensuring they are not committed to version control.

GitLab CI/CD Pipeline Configuration:

Define a GitLab CI pipeline that uses different stages for different tasks (e.g., build, test, deploy).
Use environment variables and templating to inject values dynamically based on the current environment.

Secret Management with GitLab:

Utilize GitLab's secret management features like Secret Detection and Secret Scanning.
Store secrets in a secure vault (e.g., HashiCorp Vault, AWS Secrets Manager) and use them via CI/CD pipelines.

Helm Chart Values Injection:

Use Helm’s --set command or values.yaml files to inject specific environment values at runtime.

#### Example
helm upgrade --install my-release ./my-chart --values values-production.yaml.
Detailed Steps:

Prepare Environment-Specific Value Files:

YAML
# .helm/values-testing.yaml
database:
  username: testuser
  password: testpassword

# .helm/values-production.yaml
database:
  username: produser
  password: prodpassword

GitLab CI/CD Pipeline Configuration:

YAML
stages:
  - build
  - test
  - deploy

variables:
  HELM_REPO_URL: "https://charts.example.com"
  CHART_NAME: "my-chart"

.build-job:
  stage: build
  script:
    - helm package ./charts/${CHART_NAME}
    - helm push ${CHART_NAME}-*.tgz charts/

.test-job:
  stage: test
  when: manual
  script:
    - helm dependency update ./charts/${CHART_NAME}
    - helm lint ./charts/${CHART_NAME}

.deploy-job:
  stage: deploy
  script:
    - export KUBECONFIG=${KUBECONFIG}
    - helm upgrade --install my-release ${HELM_REPO_URL}/${CHART_NAME} \
      --values .helm/values-${CI_ENVIRONMENT_NAME}.yaml
  environment:
    name: ${CI_ENVIRONMENT_NAME}

Secret Injection via GitLab CI/CD:

SHELL
# In the pipeline, use variables to inject secrets dynamically
- helm upgrade --install my-release ./my-chart \
  --set database.username=${SECRETS_DB_USERNAME} \
  --set database.password=${SECRETS_DB_PASSWORD}

Secure Secrets Management:

Use GitLab's Secret Detection and Scanning capabilities to identify secrets in code.
Store sensitive information like database credentials in a secure vault (e.g., HashiCorp Vault).

#### Closing Summary

By using environment-specific values.yaml files, GitLab CI/CD pipelines with templating, and leveraging GitLab’s secret management features, we can ensure that per-environment configurations are safely managed without exposing secrets in our code repositories. This approach ensures security, maintainability, and compliance.

#### Common Mistakes/Follow-Ups

Mistake: Committing sensitive information directly to the values.yaml files.

#### Follow-Up
Use separate environment-specific files that are not committed to version control.

Mistake: Not using GitLab's secret management features.

#### Follow-Up
Utilize built-in tools like Secret Detection and Scanning, and store secrets in secure vaults.

**Comparison:**

```
Stage	Script
build	helm package ./charts/${CHART_NAME} <br> helm push ${CHART_NAME}-*.tgz charts/
test	helm dependency update ./charts/${CHART_NAME}<br>helm lint ./charts/${CHART_NAME}
deploy	export KUBECONFIG=${KUBECONFIG}<br>helm upgrade --install my-release ${HELM_REPO_URL}/${CHART_NAME} <br> --values .helm/values-${CI_ENVIRONMENT_NAME}.yaml
```

**Code:**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  HELM_REPO_URL: "https://charts.example.com"
  CHART_NAME: "my-chart"

.build-job:
  stage: build
  script:
    - helm package ./charts/${CHART_NAME}
    - helm push ${CHART_NAME}-*.tgz charts/

.test-job:
  stage: test
  when: manual
  script:
    - helm dependency update ./charts/${CHART_NAME}
    - helm lint ./charts/${CHART_NAME}

.deploy-job:
  stage: deploy
  script:
    - export KUBECONFIG=${KUBECONFIG}
    - helm upgrade --install my-release ${HELM_REPO_URL}/${CHART_NAME} \
      --values .helm/values-${CI_ENVIRONMENT_NAME}.yaml
  environment:
    name: ${CI_ENVIRONMENT_NAME}
```

**Example input / output:**

```
Input: helm upgrade --install my-release ./my-chart --values .helm/values-testing.yaml
Output: Helm chart updated with testing environment values.
SHELL
Copy
# Example response from Helm
INFO[0003] Release "my-release" upgraded. Happy Helming!
```

**Data:**

```json
{
  "database": {
    "username": "${SECRETS_DB_USERNAME}",
    "password": "${SECRETS_DB_PASSWORD}"
  }
}
```

**Flow:**

1. CI/CD Pipeline Triggered
2. Build Stage
3. Test Stage (Manual)
4. Deploy Stage
5. Environment-Specific ValuesApplied
6. Secrets from Secure VaultInjected

