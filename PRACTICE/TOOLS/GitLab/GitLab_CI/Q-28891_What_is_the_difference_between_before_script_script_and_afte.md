---
type: "practice"
difficulty: "Basic"
---

# What is the difference between 'before_script' 'script' and 'after_script'?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Job Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/Job_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitLab CI, before_script, script, and after_script are key components that define stages in a pipeline, each serving distinct purposes. Here’s a quick overview before we dive into the details:

Before Script: This is where you run commands to prepare your environment for the main script execution.
Script: The primary section of your job definition where the actual work gets done.
After Script: Used for post-processing, like cleaning up or logging after the script phase.

Let’s break down each part in detail using first-principles reasoning.

**Comparison:**

```
Stage	Command(s)	Purpose
before_script	setup environment	Prepare the environment
script	actual job logic	Execute the main tasks
after_script	cleanup/logging	Clean up or log post-execution
```

**Code:**

```yaml
stages:
  - build

build_job:
  stage: build
  before_script:
    - echo "Setting up environment..."
  script:
    - make
  after_script:
    - mv built_binary /output_directory/
```

**Example input / output:**

```
Input: A GitLab CI/CD configuration file (.gitlab-ci.yml) with the following snippet.
YAML
Copy
stages:
  - build

build_job:
  stage: build
  before_script:
    - echo "Setting up environment..."
  script:
    - make
  after_script:
    - mv built_binary /output_directory/
Output:
before_script: The command echo "Setting up environment..." is executed first, setting the stage for subsequent commands.
script: The command make runs next in the pipeline, likely building your application.
after_script: Finally, mv built_binary /output_directory/ moves the compiled binary to a specified location.
```

