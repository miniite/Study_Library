---
type: "practice"
difficulty: "Intermediate"
---

# Your deploy freeze during a big sale needs to allow one critical fix through; how would you encode freeze windows in the pipeline and permit controlled, audited exceptions?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Emergency Deploy Paths](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Emergency_Deploy_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In this scenario, we need to handle a situation where a CI/CD pipeline is frozen during a big sale event but still allows through one critical fix. This involves understanding how freeze windows are implemented in pipelines and how controlled exceptions can be managed while maintaining auditability.

#### Triage
The question revolves around managing CI/CD pipeline freezes selectively for emergencies, ensuring that critical changes can bypass the usual constraints of the pipeline. We need to design a mechanism that can override standard checks during designated times but still maintains control and auditing over these exceptions.

#### Assumptions

The freeze is due to system-wide performance issues or other critical events.
There's a high likelihood of the sale event causing unpredictable load spikes, leading to freezes.
The organization needs to manage exceptions for non-critical versus critical changes.
We're working with an existing CI/CD pipeline that has predefined freeze windows.

#### First Principles Solution
To address this, we need to introduce a mechanism within our CI/CD tooling (e.g., Jenkins, GitLab) that can temporarily bypass certain checks during specific time periods. Additionally, the system should log and audit these exceptions for transparency and accountability.

Step-by-Step Approach

Define Freeze Windows:

Establish predefined "freeze windows" based on business needs. For example, during a sale event, freeze from 9 AM to 5 PM.

Implement Exception Handling Logic:

Create a flag or environment variable that can be set manually by authorized personnel to override the freeze.
This flag should allow for specific tasks to proceed even if the pipeline is frozen.

Integrate with CI/CD Tooling:

Modify the pipeline script to check this flag before executing any step. If the flag is set, bypass the usual checks and proceed with the task.

Audit and Logging:

Implement a logging mechanism to record every time the freeze is overridden.
Ensure that these logs are stored securely and can be audited by relevant stakeholders.

User Interface for Control:

Develop a user interface or command-line tool where authorized personnel can set and clear the override flag.

Security Measures:

Use role-based access control (RBAC) to ensure only specific users can set this flag.
Implement multi-factor authentication (MFA) for setting these overrides.

#### Example
Pipeline Configuration

Here’s an example of how you might configure a Jenkins pipeline script:

GROOVY
pipeline {
    agent any
    environment {
        FREEZE_OVERRIDE_FLAG = false // Default to false
    }

    stages {
        stage('Build') {
            when { expression { return FREEZE_OVERRIDE_FLAG || !isFrozen() } }
            steps {
                echo 'Building the application...'
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            when { expression { return FREEZE_OVERRIDE_FLAG || !isFrozen() } }
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            when { expression { return FREEZE_OVERRIDE_FLAG || !isFrozen() } }
            steps {
                echo 'Deploying to production...'
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }

    post {
        always {
            // Logging and audit
            archiveArtifacts artifacts: '**/build.log', allowEmptyArchive: true, fingerprint: true
        }
    }
}

def isFrozen() {
    // Logic to check if the pipeline should be frozen based on current time or other criteria
    return (System.currentTimeMillis() >= freezeWindowStart && System.currentTimeMillis() <= freezeWindowEnd)
}

#### Closing Summary
and Common Mistakes

This approach ensures that critical changes can still be deployed even during a freeze window. By using environment variables, we maintain the flexibility to control these exceptions selectively. However, it’s crucial to manage this mechanism carefully to avoid security vulnerabilities or accidental bypasses.

#### Common Mistakes

Overlooking proper logging and auditing.
Not implementing strong access controls for setting flags.
Failing to document and train stakeholders on how to use the override feature.
Ignoring edge cases where the freeze logic might not be properly checked (e.g., timezone differences).

**Comparison:**

```
Stage	When Condition	Action
Build	FREEZE_OVERRIDE_FLAG
Test	FREEZE_OVERRIDE_FLAG
Deploy	FREEZE_OVERRIDE_FLAG
```

**Code:**

```
GROOVY
Copy
pipeline {
    agent any
    environment {
        FREEZE_OVERRIDE_FLAG = false // Default to false
    }

    stages {
        stage('Build') {
            when { expression { return FREEZE_OVERRIDE_FLAG || !isFrozen() } }
            steps {
                echo 'Building the application...'
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            when { expression { return FREEZE_OVERRIDE_FLAG || !isFrozen() } }
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            when { expression { return FREEZE_OVERRIDE_FLAG || !isFrozen() } }
            steps {
                echo 'Deploying to production...'
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }

    post {
        always {
            // Logging and audit
            archiveArtifacts artifacts: '**/build.log', allowEmptyArchive: true, fingerprint: true
        }
    }
}

def isFrozen() {
    // Logic to check if the pipeline should be frozen based on current time or other criteria
    return (System.currentTimeMillis() >= freezeWindowStart && System.currentTimeMillis() <= freezeWindowEnd)
}
```

**Example input / output:**

```
Input: Set FREEZE_OVERRIDE_FLAG to true and trigger the pipeline.
Output:
CODE
Copy
[Pipeline] node
Running on Jenkins in /var/jenkins/workspace/sale_pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Build)
Building the application...
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test)
Running tests...
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy)
Deploying to production...
[Pipeline] }
[Pipeline] // stage
[Pipeline] post
[Pipeline] always
[Pipeline] archiveArtifacts
Archiving artifacts
```

**Data:**

```text
- `freezeWindowStart` and `freezeWindowEnd`: Timestamps defining the start and end of the freeze window.
- Example: `freezWindowStart = 1628340800000L; freezWindowEnd = 1628359200000L`.
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Freez Window Active?
5. Log Exception and Allow Task
6. Execute Next Stage
7. Archive Artifacts

