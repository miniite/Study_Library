---
type: "practice"
difficulty: "Deep"
---

# You need a rollback job that reverts to the previous image. How would you design it and track the last known good tag?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Debugging Runner Failures](<../../../../TOOLS/GitLab/GitLab_CI/Debugging_Runner_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of implementing a rollback job in GitLab CI, we need to ensure that our pipeline can revert to the previous Docker image when needed. This involves tracking the last known good tag and ensuring the rollback process is reliable and secure.

TL;DR:

Use GitLab CI/CD to define a custom job for rollback.
Track the last known good tag using environment variables or a database.
Implement a strategy to safely revert the application in stages, including rolling back both code and Docker images.

#### Triaging the Question

The question is asking how to design and implement a rollback mechanism that can revert to a previous image tagged in a GitLab project. The interviewer is probing your understanding of GitLab CI/CD, versioning strategies, and robust rollback procedures. Key assumptions include:

We are working with Docker images.
The application uses Git for version control.
There is a need to reliably track the last known good tag.
Step-by-Step Solution

Environment Variables or Database Tracking:

Use environment variables in your .gitlab-ci.yml file to store the latest and previous tags.
YAML
variables:
  LAST_KNOWN_GOOD_TAG: "v1.0"

Custom CI Job for Rollback:

Define a custom job that will be triggered by specific conditions, such as a tag or a pipeline failure.
YAML
rollback_job:
  stage: rollback
  script:
    - echo "Rolling back to the previous image..."
    - docker pull registry.example.com/myapp/${!LAST_KNOWN_GOOD_TAG}
    - docker stop myapp  # Stop running containers if necessary
    - docker rm myapp    # Remove old containers
    - docker run -d --name myapp ${!LAST_KNOWN_GOOD_TAG}  # Re-run with the previous image
  only:
    refs:
      - tags  # Trigger on tag events

Handling Edge Cases:

Ensure that the rollback job handles edge cases, such as multiple running instances or incomplete rollbacks.

#### Cons
ider using a database to track not just tags but also other relevant metadata like timestamps.

Rolling Back Multiple Stages:

If your application has multiple stages (e.g., frontend and backend), ensure that the rollback covers all dependencies.
YAML
rollback_job:
  stage: rollback
  script:
    - echo "Rolling back to the previous image..."
    - docker stop myapp  # Stop running containers if necessary
    - docker rm myapp    # Remove old containers
    - docker run -d --name myapp ${!LAST_KNOWN_GOOD_TAG}  # Re-run with the previous image
    - ./scripts/rollback_frontend.sh v1.0   # Rollback frontend as well
  only:
    refs:
      - tags  # Trigger on tag events

Testing and Validation:

Regularly test your rollback mechanism to ensure it works without issues.
Use a staging environment for testing before deploying changes.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Ensure that the rollback process is reliable, secure, and follows best practices.

#### Cons
ider adding automated tests to validate the rollback process.
Discuss how you would integrate this with other CI/CD pipelines, such as those involving multiple microservices or container orchestration tools.

#### Common mistakes
include:

Not tracking tags effectively, leading to confusion during rollbacks.
Failing to handle edge cases properly, resulting in incomplete rollbacks.
Ignoring security concerns, such as ensuring that the rollback mechanism cannot be exploited by malicious actors.
Common Follow-Ups
How would you ensure the rollback process is secure and cannot be abused?
What are some best practices for tracking tags and managing multiple environments (e.g., dev, staging, prod)?

**Comparison:**

```
Stage	Script
rollback	echo "Rolling back to the previous image..."<br>docker pull registry.example.com/myapp/${!LAST_KNOWN_GOOD_TAG}<br>docker stop myapp<br>docker rm myapp<br>docker run -d --name myapp ${!LAST_KNOWN_GOOD_TAG}
```

**Code:**

```yaml
variables:
  LAST_KNOWN_GOOD_TAG: "v1.0"

rollback_job:
  stage: rollback
  script:
    - echo "Rolling back to the previous image..."
    - docker pull registry.example.com/myapp/${!LAST_KNOWN_GOOD_TAG}
    - docker stop myapp  # Stop running containers if necessary
    - docker rm myapp    # Remove old containers
    - docker run -d --name myapp ${!LAST_KNOWN_GOOD_TAG}  # Re-run with the previous image
  only:
    refs:
      - tags
```

**Example input / output:**

```text
# Sample input
variables:
  LAST_KNOWN_GOOD_TAG: "v1.0"

rollback_job:
  stage: rollback
  script:
    - echo "Rolling back to the previous image..."
    - docker pull registry.example.com/myapp/${!LAST_KNOWN_GOOD_TAG}
    - docker stop myapp  # Stop running containers if necessary
    - docker rm myapp    # Remove old containers
    - docker run -d --name myapp ${!LAST_KNOWN_GOOD_TAG}  # Re-run with the previous image
  only:
    refs:
      - tags

# Sample output
Rolling back to the previous image...
Pulling v1.0 from registry.example.com/myapp...
Stopping myapp...
Removing myapp...
Running v1.0
```

**Data:**

```yaml
# Example of a .gitlab-ci.yml snippet
variables:
  LAST_KNOWN_GOOD_TAG: "v1.0"

rollback_job:
  stage: rollback
  script:
    - echo "Rolling back to the previous image..."
    - docker pull registry.example.com/myapp/${!LAST_KNOWN_GOOD_TAG}
    - docker stop myapp  # Stop running containers if necessary
    - docker rm myapp    # Remove old containers
    - docker run -d --name myapp ${!LAST_KNOWN_GOOD_TAG}  # Re-run with the previous image
  only:
    refs:
      - tags
```

**Flow:**

1. Yes
2. Define CI Job
3. Is Tag Event?
4. UpdateLAST_KNOWN_GOOD_TAG
5. Rollback Job Triggered
6. RUN ROLLBACK SCRIPT

