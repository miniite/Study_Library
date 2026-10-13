---
type: "practice"
difficulty: "Basic"
---

# What is an exit code and how does a pipeline use it to decide if a step passed or failed?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline as Code](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_as_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Exit codes are numerical values that processes return to indicate whether they completed successfully, with 0 typically signifying success, and non-zero values representing different failure conditions. In the context of CI/CD pipelines, exit codes play a crucial role in determining the outcome of each step in the pipeline.

First, let's understand what an exit code is (TL;DR). An exit code is like a signal that tells you if something went right or wrong when running a command or script. Now, how do CI/CD pipelines use this to make decisions? Pipelines typically run commands and scripts as part of their build or deployment process. If these commands complete without errors, they return an exit code of 0. Any non-zero value indicates that the command encountered some issue during execution.

Now, let's break it down step-by-step from first principles:

Command Execution: Each step in a CI/CD pipeline runs a specific command or script.
Exit Code Check: After each command completes, its exit code is checked by the pipeline.
Decision Making: Based on the exit code, the pipeline makes decisions about whether to proceed to the next step or halt.

For example, if we have a shell script that compiles Java code:

BASH
# Compile Java Code
mvn compile

# Check Exit Code
if [ $? -eq 0 ]; then
    echo "Compilation succeeded."
else
    echo "Compilation failed. Exiting pipeline."
    exit 1
fi

In this example, ? is a special variable in shell scripts that holds the last command's exit code. If the compilation fails (exit code non-zero), the script exits with an error (exit code 1) to signal failure to the CI/CD system.

In more complex pipelines, multiple commands might be chained together using tools like Jenkins, GitLab CI, or GitHub Actions. For instance, a pipeline step that installs dependencies and runs tests would look something like this:

YAML
steps:
- name: Install Dependencies
  run: npm install

- name: Run Tests
  run: npm test
  if: ${{ failure() && steps.InstallDependencies.outcome == 'success' }}

Here, the if condition checks if the previous step (installing dependencies) was successful and only runs the tests if that is the case. If any of these commands fail, the pipeline will stop and mark the build as failed.

#### Closing Summary
Exit codes are critical in CI/CD pipelines for determining whether a command or script executed successfully. By using exit codes, pipelines can make informed decisions about proceeding to the next steps or stopping execution entirely based on predefined conditions.

#### Common Mistakes/Follow-ups

Ignoring Exit Codes: One common mistake is ignoring exit codes and allowing pipelines to proceed even when critical commands fail.
Misinterpreting Exit Codes: Another issue is misinterpreting what a non-zero exit code means, which can lead to incorrect decision-making in the pipeline.

**Comparison:**

```
FLOWCHART · MERMAID
graph LR;
    A[Install Dependencies] --> B(Run Tests);
    B --> C(Failure Detected) [Exit Code != 0];
    C --> D[Pipeline Halted];
    D --> E[Report Failure];
    B --> F(Pipeline Proceeds) [Exit Code == 0];
```

**Code:**

```bash
# Example Shell Script for CI/CD Step
mvn compile
if [ $? -ne 0 ]; then
    echo "Compilation failed. Exiting."
    exit 1
fi

npm install
if [ $? -ne 0 ]; then
    echo "Node.js dependency installation failed. Exiting."
    exit 1
fi

npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Exiting."
    exit 1
fi
```

**Example input / output:**

| Command | Exit Code | Result |
|---------|-----------|--------|
| `mvn compile` | 0 | Compilation succeeded. |
| `npm install` | 1 | Installation failed due to missing package. Pipeline halted. |
| `npm test` | 0 | All tests passed. |

**Data:**

```yaml
# Example YAML for GitLab CI/CD Pipeline
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - mvn compile
    - npm install

test-job:
  stage: test
  script:
    - npm test
```

**Flow:**

1. Yes
2. No
3. Run Command
4. Exit Code == 0?
5. Proceed to Next Step
6. Halt Pipeline & Log Error
7. Log Success
8. Report Failure and Halt

