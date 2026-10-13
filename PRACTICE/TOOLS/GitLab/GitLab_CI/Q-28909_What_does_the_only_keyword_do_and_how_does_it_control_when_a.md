---
type: "practice"
difficulty: "Intermediate"
---

# What does the 'only' keyword do and how does it control when a job runs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Job Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/Job_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitLab CI, the only keyword is used to specify conditions under which a job should be executed. It's particularly useful for controlling the build process based on certain environments or branches, ensuring that not all jobs are run in every pipeline execution. The only keyword works by specifying an array of values, and if any one of them matches the current context, the job will execute.

The key idea is that only allows you to fine-tune your CI/CD pipelines, making sure that certain steps only occur under specific conditions. This can help in reducing unnecessary build times and ensuring that sensitive tasks are only run when necessary.

**Code:**

```yaml
# Example GitLab CI configuration with 'only' keyword
stages:
  - build

build_job:
  stage: build
  script:
    - echo "Building the application..."
  only:
    - master
    - tags:~alpha*
```

**Example input / output:**

```

In this example, the build_job will run on any commit to the master branch and exclude commits tagged with alpha*. The job won't run for other branches or tags.

YAML
Copy
# Example input-output scenario
stages:
  - build

build_job:
  stage: build
  script:
    - echo "Building the application..."
  only:
    - master
    - tags:~alpha*
```

**Flow:**

1. Yes
2. No
3. Job Configuration
4. Define `only` conditions
5. Condition Match?
6. RUN job
7. Do NOT RUN job

