---
type: "practice"
difficulty: "Basic"
---

# What happens to the pipeline if a job in an earlier stage fails?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Debugging Runner Failures](<../../../../TOOLS/GitLab/GitLab_CI/Debugging_Runner_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a job in an earlier stage of a GitLab CI pipeline fails, the immediate and automatic consequence is that subsequent jobs in later stages will not be executed. This behavior ensures that only successful builds move forward to the next steps, maintaining the integrity of your build process.

To triage this question, we need to understand what "stages" and "jobs" mean in GitLab CI context:

Stages: These are logical groups within a pipeline that represent different phases of your build process (e.g., build, test, deploy).
Jobs: These are the individual tasks or steps defined under each stage. Each job is an executable script or command that runs in a container.

In GitLab CI, when a job fails:

The execution of the current pipeline stops.
The failed job's logs and status are available for review.
The downstream jobs (jobs in subsequent stages) do not run.

For example, if you have a build stage followed by a test stage, and the build stage fails due to an error, the test stage will not be executed. This helps in isolating issues early in your pipeline, ensuring that only stable code proceeds to critical stages like deployment.

To reason from first principles:

GitLab CI is designed with a robust failure strategy; if one job fails, it prevents further execution to avoid propagating errors downstream.
The configuration for this behavior can be controlled by setting appropriate rules in your pipeline's YAML definition. For instance, using the on_success: continue and on_failure: continue rules allows more flexibility but is not the default behavior.

In summary, a failed job in GitLab CI stops the execution of subsequent jobs within its stage, ensuring that only successful stages proceed to further steps. This design promotes robust testing and deployment practices by isolating failing builds early on.

#### Common mistakes
include overlooking the immediate stopping of downstream jobs or misunderstanding how rules can modify this default behavior.

**Code:**

```yaml
# Example GitLab CI YAML configuration

stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "Building application"
    - exit 1 # Intentionally failing for demonstration

test-job:
  stage: test
  dependencies:
    - build-job
  script:
    - echo "Running tests"
```

