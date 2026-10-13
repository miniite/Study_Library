---
type: "practice"
difficulty: "Deep"
---

# How would you fail a build when a new dependency introduces a disallowed software license?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deployment Gating Trade-offs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how we might fail a build when a new dependency introduces an unapproved software license, let's first summarize the core issue and approach it from first principles.

TL;DR:

The goal is to ensure that no builds proceed if any of the newly added dependencies violate our organization’s policy on acceptable licenses. This involves integrating static analysis tools into our CI/CD pipeline to scan for disallowed licenses before allowing the build to complete successfully.

#### Triaging the Question

This question probes your understanding of how to enforce licensing policies within a CI/CD environment, specifically focusing on proactive security measures and integration with dependency management systems. Key assumptions include:

The organization has defined a list of allowed software licenses.
Dependency scanning tools are available for use in the CI/CD pipeline.
There is a need for real-time or near-real-time license validation.

#### First-Principles Solution

Define Allowed Licenses:

Create a whitelist of approved licenses based on your organization's policy.

Integrate License Scanning Tools:

Use tools like Snyk, WhiteSource, or Black Duck to scan the dependency tree.
These tools can integrate with popular package managers (npm, Maven, etc.) and provide comprehensive reports.

Configure CI/CD Pipeline:

Modify your CI pipeline configuration files (Jenkinsfile, GitLabCI YAML) to include steps for license scanning.

#### Example
snippet in a Jenkinsfile:
GROOVY
stage('Dependency Check') {
    steps {
        script {
            sh 'snyk scan --test'
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/snyk-report.json', allowEmptyArchive: true
        }
        failure {
            echo "Build failed because of unapproved licenses."
        }
    }
}

Automate and Monitor:

Ensure that the license scanning step runs automatically with every build.
Set up alerts or notifications if any disallowed licenses are detected.

Review and Validate:

Periodically review the integration to ensure it is working as expected.
Test edge cases like partial dependencies, transitive dependencies, etc.

#### Closing Summary

By integrating a robust license scanning tool into our CI/CD pipeline, we can proactively prevent builds that include unapproved software licenses. This not only ensures compliance with organizational policies but also enhances the security posture of the application by identifying and addressing potential vulnerabilities early in the development lifecycle.

#### Common Mistakes
and Follow-Ups
Failing to Configure the Tool Properly: Ensure all dependencies are correctly identified, including transitive ones.
Overlooking Transitive Dependencies: Disallowed licenses might be introduced through indirect dependencies. Tools like snyk org upgrade can help manage this.
Not Setting Up Alerts: Failure to set up notifications for license violations can result in unnoticed issues.

Discussing these points and any specific tools or processes your organization uses will demonstrate a deep understanding of the problem and its solution.

**Comparison:**

```
License	Status
MIT	Failed
Apache	Passed
BSD	Passed
License ID	Description	Severity
SNYK-JS-GITHUBCOM-TYPESCRIPTX-123456	MIT license is not allowed	High
```

**Code:**

```
GROOVY
Copy
// Example Jenkinsfile snippet for dependency check
stage('Dependency Check') {
    steps {
        script {
            sh 'snyk scan --test'
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/snyk-report.json', allowEmptyArchive: true
        }
        failure {
            echo "Build failed because of unapproved licenses."
        }
    }
}
```

**Example input / output:**

```

Input: A new dependency introduced in the project that includes a disallowed MIT license.

Output: The CI/CD pipeline fails with an error message and logs indicating the presence of unapproved licenses.

Example Response from Snyk Scan:

CODE
Copy
[INFO] Detected dependencies with known vulnerabilities.
[ERROR] SNYK-JS-GITHUBCOM-TYPESCRIPTX-123456: MIT license is not allowed (high severity).
[INFO] Build failed due to unapproved licenses.
```

**Data:**

```yaml
# Example Snyk report snippet
version: 1.0.4
status: error
message: Detected a disallowed license in project dependencies.
rules:
- id: SNYK-JS-GITHUBCOM-TYPESCRIPTX-123456
  severity: high
  title: MIT license is not allowed
  cve: N/A
```

**Flow:**

1. CI/CD Pipeline
2. Dependency Check
3. License Scanning Tool (e.g.,Snyk)
4. Report Generation
5. Fail Build if Disallowed LicenseDetected
6. Send Alert to Dev Team

