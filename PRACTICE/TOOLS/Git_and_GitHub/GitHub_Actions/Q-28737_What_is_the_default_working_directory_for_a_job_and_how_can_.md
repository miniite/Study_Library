---
type: "practice"
difficulty: "Intermediate"
---

# What is the default working directory for a job and how can you change it for a step?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Self-hosted Runners](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Self-hosted_Runners.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitHub Actions, understanding the default working directory for jobs and how to change it is crucial for managing your workflows effectively. By default, each job runs in its own working directory which is the root of the repository where you've pushed your code. However, sometimes you might need a different context or directory structure for certain steps within your workflow.

#### Triage
What the Question is Asking and Assumptions

The question asks about the default working directory (WD) for jobs in GitHub Actions and how to customize it. Here are some key points:

Default WD: Each job runs from the root of the repository.
Customization: You can change this WD using specific commands or parameters.

#### First-Principles Solution

GitHub Actions use a YAML configuration file (e.g., .github/workflows/*.yml) to define workflows. Jobs are defined within these files, and each job has its own working directory by default. However, you can explicitly set the working-directory field at the job level or the step level.

Step-by-Step Solution

Define a Job:

YAML
jobs:
  build:
    runs-on: ubuntu-latest

Set Default Working Directory for Job (Optional): If you want to set a default working directory at the job level, use the working-directory field.

YAML
jobs:
  build:
    runs-on: ubuntu-latest
    working-directory: ./src

Change Working Directory for a Step: You can override the default WD for specific steps within a job by using the working-directory key in step definitions.

YAML
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Change Working Directory
        run: echo "Current working directory is $PWD"
        working-directory: ./src

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the default working directory for a GitHub Actions job is the root of the repository. You can change this by using the working-directory field at both the job and step levels. Remember that if you override it in steps, subsequent steps will run from their respective directories unless explicitly overridden.

#### Common mistakes
include:

Forgetting to set the correct working directory, leading to commands failing because they are executed relative to the wrong path.
Not understanding how the working-directory affects all steps within a job versus just certain steps.

For follow-ups, you might be asked about the implications of changing directories for performance or security, or whether it's better to structure your repository differently to avoid such issues.

**Code:**

```yaml
# Example workflow with custom working directory
jobs:
  build:
    runs-on: ubuntu-latest
    working-directory: ./src

    steps:
      - name: Change Working Directory
        run: echo "Current working directory is $PWD"
        working-directory: ./subdir

      - name: Another Step
        run: ls
```

