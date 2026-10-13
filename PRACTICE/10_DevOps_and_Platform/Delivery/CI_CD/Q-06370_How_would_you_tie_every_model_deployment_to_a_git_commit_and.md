---
type: "practice"
difficulty: "Deep"
---

# How would you tie every model deployment to a git commit and a config hash so that when quality drops in prod, you can trace it to the exact prompt+model+retriever version that shipped?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Build and Release Metadata](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that every model deployment is tied back to a specific git commit and config hash, we need to establish a robust CI/CD pipeline that integrates source code management, build automation, and logging mechanisms. This setup will help us trace quality issues directly to the exact version deployed in production, enabling rapid troubleshooting and rollback if necessary.

#### Triage
Understanding the Question

The core of this question revolves around ensuring transparency and traceability in model deployments within a CI/CD pipeline. The key is to correlate changes in both the codebase (git commits) and configuration settings with the models deployed into production. This requires integrating version control systems, automation tools, and logging practices.

#### First Principles Solution
Git Commit Integration: Each deployment must be tied to a specific git commit hash.
Config Hash Tracking: Configuration files should have their own unique hashes that are tracked alongside deployments.
Continuous Integration/Continuous Deployment (CI/CD) Pipeline: This pipeline will manage the build, test, and deployment processes, ensuring consistency and reliability.
Post-Deployment Logging: Deployments need to be logged with detailed information about changes made.
Step-by-Step Solution

Source Code Management:

Ensure that all model-related code is stored in a version control system like Git.
Commit changes regularly, including updates to models and configurations.

Build Automation:

Use tools like Jenkins, GitLab CI/CD, or GitHub Actions to automate the build process.
Each deployment job should start with fetching the latest commit from the specified branch or tag.

Configuration Tracking:

Store configuration files in version control alongside code.
Before each deployment, generate a hash of the current configuration file(s).
Store these hashes as environment variables or metadata in your build artifacts.

Deployment Pipeline:

Implement a CI/CD pipeline that triggers on git push events or manual deployments.
The pipeline should include steps to:
Checkout the correct commit from Git.
Build the model and configurations using the fetched code.
Generate and log config hashes.
Deploy the built artifacts to production.

Post-Deployment Logging:

Log all deployment details, including git commit hash, config hashes, timestamp, and other relevant metadata.
Store these logs in a centralized logging system like ELK Stack or Splunk.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By following this approach, you ensure that every model deployment is traceable to its exact source code and configuration. This will help quickly identify the root cause of any quality issues detected in production. Common mistakes include not properly versioning configurations, neglecting to log important details, or lacking a robust CI/CD setup. Follow-ups might involve discussing how to handle rollback scenarios, implementing automated testing, or integrating with monitoring tools.

**Comparison:**

| Stage | Action |
|-------|--------|
| Build | Fetch commit, build artifacts |
|       | Generate config hashes          |
| Deploy| Log deployment details         |
|       | Perform actual deployment      |

**Code:**

```python
# Example Jenkinsfile snippet for GitLab CI/CD
stages {
    'build' {
        stage('Build') {
            script {
                git_commit_hash = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                config_hashes = []
                configFileNames.each { fileName ->
                    config_hashes << sh(script: "sha256sum ${fileName}", returnStdout: true)
                }
                env.GIT_COMMIT_HASH = git_commit_hash
                env.CONFIG_HASHES = config_hashes.join(',')
            }
        }
    }
    'deploy' {
        stage('Deploy') {
            script {
                echo "Deploying commit: $env.GIT_COMMIT_HASH"
                echo "Config hashes: $env.CONFIG_HASHES"
                // Deploy logic here
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
- Git commit hash: abc1234567890
- Config hashes: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855,d41d8cd98f00b204e9800998ecf8427e

Output:
- Deployed commit: abc1234567890
- Config hashes: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855,d41d8cd98f00b204e9800998ecf8427e
```

**Data:**

```json
{
  "git_commit_hash": "abc1234567890",
  "config_hashes": [
    "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "d41d8cd98f00b204e9800998ecf8427e"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Git Commit & Config Hash
4. Are changes committed?
5. Fetch latest commit
6. Generate config hashes
7. Build model andconfigurations
8. Log deployment details
9. Deploy to production
10. End

