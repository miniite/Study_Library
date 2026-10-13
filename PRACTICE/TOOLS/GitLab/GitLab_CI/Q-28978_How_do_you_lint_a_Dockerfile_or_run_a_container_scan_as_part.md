---
type: "practice"
difficulty: "Intermediate"
---

# How do you lint a Dockerfile or run a container scan as part of your pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Docker Layer Caching](<../../../../TOOLS/GitLab/GitLab_CI/Docker_Layer_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your Dockerfiles are free of syntax errors and to detect potential security vulnerabilities, you can integrate Dockerfile linters and container image scanning into your GitLab CI pipeline. This process involves creating a .gitlab-ci.yml configuration file that runs specific tasks during the build stage.

#### Triaging the Question

The question is essentially asking how to incorporate static analysis tools for Dockerfiles (linters) and dynamic security checks (container scans) within a continuous integration pipeline managed by GitLab CI. The interviewer wants you to demonstrate your understanding of GitLab CI workflows, configuration files, and common practices in DevOps.

#### First-Principles Solution
Set Up Linter: Use a Dockerfile linter such as dfgrep or dockfmt.
Configure GitLab CI: Write a .gitlab-ci.yml file to include stages for linting and scanning.
Define Jobs: Create separate jobs in the pipeline for each task (linting, scanning).
Integrate Tools: Utilize Docker images from tool vendors that offer these functionalities.
Full Solution
Step 1: Install Linter and Scanner Tools

First, ensure you have a Dockerfile linter like dfgrep installed in your development environment or in the GitLab CI runners. Similarly, for container image scanning, tools such as Clair, Trivy, or Aqua Security can be used.

BASH
# Example of installing dfgrep and trivy on a runner
apk add --no-cache dfgrep
Step 2: Create .gitlab-ci.yml Configuration

Next, create the .gitlab-ci.yml file to define the pipeline stages and jobs. Here’s an example configuration:

YAML
# .gitlab-ci.yml
stages:
  - lint
  - scan

lint-dockerfile:
  stage: lint
  script:
    - dfgrep <path_to_dockerfile>
  artifacts:
    reports:
      txt: <output_file>

scan-container:
  stage: scan
  image: aquasec/trivy:latest
  script:
    - trivy --exit-code 1 --severity CRITICAL, HIGH, MEDIUM <image_name>
Step 3: Define Jobs and Artifacts
lint-dockerfile: This job runs dfgrep on the Dockerfile to check for syntax errors.
scan-container: Uses the Trivy image scanner to perform a security scan of the container image.

#### Closing Summary

By integrating these tools into your GitLab CI pipeline, you can ensure that your Dockerfiles are well-formed and that your container images do not contain known vulnerabilities. This approach enhances code quality and security posture in your application deployment process.

#### Common Mistakes/Follow-Ups
Using the Wrong Linter or Scanner: Ensure you choose tools appropriate for your environment.
Misconfiguring Artifacts: Incorrectly setting up artifact paths can result in losing important diagnostic information.
Ignoring Scan Results: It's crucial to handle the output of these scans and take corrective actions based on findings.

Do you have any follow-up questions or need more details on a specific aspect?

**Comparison:**

```
Job Name	Stage	Script	Artifacts
lint-dockerfile	lint	dfgrep Dockerfile	output.txt
scan-container	scan	trivy --exit-code 1 --severity CRITICAL, HIGH, MEDIUM my-image:latest	N/A
```

**Code:**

```yaml
stages:
  - lint
  - scan

lint-dockerfile:
  stage: lint
  script:
    - dfgrep Dockerfile
  artifacts:
    reports:
      txt: output.txt

scan-container:
  stage: scan
  image: aquasec/trivy:latest
  script:
    - trivy --exit-code 1 --severity CRITICAL, HIGH, MEDIUM my-image:latest
```

**Example input / output:**

```
Input: A Dockerfile containing syntax errors and a sample container image.
Output:
dfgrep produces an output file named output.txt.
Trivy scans the image and outputs potential security vulnerabilities.
PLAINTEXT
Copy
$ dfgrep Dockerfile
Syntax error: Invalid line in Dockerfile

$ trivy --exit-code 1 --severity CRITICAL, HIGH, MEDIUM my-image:latest
Found 3 high severity issues.
```

**Data:**

```json
{
  "stages": ["lint", "scan"],
  "lint-dockerfile": {
    "stage": "lint",
    "script": [
      "dfgrep Dockerfile"
    ],
    "artifacts": {
      "reports": {
        "txt": "output.txt"
      }
    }
  },
  "scan-container": {
    "stage": "scan",
    "image": "aquasec/trivy:latest",
    "script": [
      "trivy --exit-code 1 --severity CRITICAL, HIGH, MEDIUM my-image:latest"
    ]
  }
}
```

**Flow:**

1. CI Pipeline
2. Lint Stage
3. Lint Dockerfile
4. Linter Output
5. Dockerfile Validation
6. Build Stage
7. Scan Stage
8. Image Scan
9. Vulnerability Report
10. Security Assessment

