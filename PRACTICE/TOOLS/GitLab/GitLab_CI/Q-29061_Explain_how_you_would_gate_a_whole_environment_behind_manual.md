---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would gate a whole environment behind manual approval only for production while auto-deploying staging.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Environments](<../../../../TOOLS/GitLab/GitLab_CI/Protected_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that deployments to the production environment require explicit human intervention via manual approval, whereas staging environments can be deployed automatically. This setup is crucial for maintaining a high level of security and control over the production environment without disrupting the automated deployment process in staging.

To achieve this, we'll configure GitLab CI/CD pipelines with specific settings for both the staging and production environments. The key steps involve:

Setting up distinct stages for each environment.
Configuring the before_script and script sections to include the necessary commands for deployments.
Implementing a custom GitLab runner that can only approve builds in the production stage manually.

#### Triaging the Question

The interviewer is probing our understanding of GitLab CI/CD, pipeline management, and automation controls. We need to consider how to balance automated workflows with manual approvals, ensuring that critical environments like production are not automatically deployed without human oversight.

#### Assumptions
There are separate GitLab projects for staging and production.
The environment variables and secrets required for deployments are properly set up in the respective GitLab projects.
We have a GitLab runner or runners capable of executing our CI/CD pipelines.
We're using GitLab's built-in mechanisms to control pipeline execution.
Step-by-Step Solution

Project Setup:

Ensure you have two separate GitLab projects: one for staging and one for production.

Staging Pipeline Configuration (.gitlab-ci.yml):

YAML
stages:
  - build
  - test
  - deploy_staging
  - deploy_production

variables:
  DEPLOY_ENV: "staging"

before_script:
  - echo "Executing before script for $DEPLOY_ENV"
  # Add any necessary setup commands here, e.g., database migrations, tests.

script:
  - echo "Running deployment script for $DEPLOY_ENV"
  # Add the deployment commands specific to staging.

deploy_staging:
  stage: deploy_staging
  when: on_success
  except:
    - production
  script:
    - echo "Deploying to staging environment"
    - ./deploy.sh

deploy_production:
  stage: deploy_production
  when: manual
  only:
    - production
  script:
    - echo "Manual approval required for deployment to production"
    # Add the deployment commands specific to production.

Production Pipeline Configuration (.gitlab-ci.yml):

YAML
stages:
  - build
  - test
  - deploy_staging
  - deploy_production

variables:
  DEPLOY_ENV: "production"

before_script:
  - echo "Executing before script for $DEPLOY_ENV"
  # Add any necessary setup commands here, e.g., database migrations, tests.

script:
  - echo "Running deployment script for $DEPLOY_ENV"
  # Add the deployment commands specific to production.

deploy_staging:
  stage: deploy_staging
  when: on_success
  except:
    - production
  script:
    - echo "Deploying to staging environment"
    - ./deploy.sh

deploy_production:
  stage: deploy_production
  when: manual
  only:
    - production
  script:
    - echo "Manual approval required for deployment to production"
    # Add the deployment commands specific to production.

Custom GitLab Runner Configuration: Ensure that the runner used for the deploy_production stage is configured to require manual intervention. You can achieve this by setting up the appropriate settings in your GitLab project's CI/CD pipeline configuration.

Run and Monitor Pipelines:

Deploy changes to both environments using their respective pipelines.
Verify that staging deployments proceed automatically while production deployments require explicit approval from a designated approver.

#### Closing Summary

By configuring distinct stages for automatic deployment in the staging environment and manual approval in the production environment, we can effectively control when and how our applications are deployed. This approach ensures that critical changes to the production environment undergo thorough review before being pushed live.

#### Common Mistakes
and Follow-Ups
Common Mistake: Forgetting to set when: manual for the production deployment stage.

#### Follow-Up
Discussing scenarios where automatic deployments might be acceptable in non-production environments, such as using canary releases or blue-green deployments.

**Comparison:**

```
Stage	Environment	When	Only
build	Both	always	-
test	Both	on_success	-
deploy_staging	staging	on_success	except: production
deploy_production	production	manual	only: production
Example of a simple table showing the different stages and their configuration
```

**Code:**

```yaml
# Staging .gitlab-ci.yml
stages:
  - build
  - test
  - deploy_staging
  - deploy_production

variables:
  DEPLOY_ENV: "staging"

before_script:
  - echo "Executing before script for $DEPLOY_ENV"
  # Add any necessary setup commands here, e.g., database migrations, tests.

script:
  - echo "Running deployment script for $DEPLOY_ENV"
  # Add the deployment commands specific to staging.

deploy_staging:
  stage: deploy_staging
  when: on_success
  except:
    - production
  script:
    - echo "Deploying to staging environment"
    - ./deploy.sh

deploy_production:
  stage: deploy_production
  when: manual
  only:
    - production
  script:
    - echo "Manual approval required for deployment to production"
    # Add the deployment commands specific to production.

# Production .gitlab-ci.yml
stages:
  - build
  - test
  - deploy_staging
  - deploy_production

variables:
  DEPLOY_ENV: "production"

before_script:
  - echo "Executing before script for $DEPLOY_ENV"
  # Add any necessary setup commands here, e.g., database migrations, tests.

script:
  - echo "Running deployment script for $DEPLOY_ENV"
  # Add the deployment commands specific to production.

deploy_staging:
  stage: deploy_staging
  when: on_success
  except:
    - production
  script:
    - echo "Deploying to staging environment"
    - ./deploy.sh

deploy_production:
  stage: deploy_production
  when: manual
  only:
    - production
  script:
    - echo "Manual approval required for deployment to production"
    # Add the deployment commands specific to production.
```

**Example input / output:**

```text
# Example of a manual approval request in GitLab CI/CD pipeline logs
Deploying to production environment
Manual approval required for deployment to production
```

**Data:**

```yaml
# Example of a .gitlab-ci.yml configuration snippet
stages:
  - build
  - test
  - deploy_staging
  - deploy_production

variables:
  DEPLOY_ENV: "production"

before_script:
  - echo "Executing before script for $DEPLOY_ENV"
  # Add any necessary setup commands here, e.g., database migrations, tests.

script:
  - echo "Running deployment script for $DEPLOY_ENV"
  # Add the deployment commands specific to production.
```

**Flow:**

1. Yes
2. No
3. Build
4. Test
5. Deploy to Staging?
6. Staging Deployed
7. Manual Approval Required forProduction
8. Production Deployed

