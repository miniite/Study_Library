---
type: "practice"
difficulty: "Basic"
---

# How do you notify a team about pipeline failures (Slack, email, etc.)?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pipeline Triggers and Path Filters](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_Triggers_and_Path_Filters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively notify a team about CI/CD pipeline failures, we need to establish a robust mechanism that ensures immediate and clear communication of issues. This involves setting up integrations between our CI/CD tools (like Jenkins, GitLab CI, or GitHub Actions) with external notification services such as Slack or email.

#### Triage

The core question here is how to efficiently integrate CI/CD failure notifications into existing workflows. The interviewer might be probing on:

Integration setup and configuration
Notification types (Slack vs. Email)
Customization of messages
Handling different severity levels

#### First Principles Solution
Identify the CI/CD Tool: Determine which CI/CD tool is being used in your project.
Define Notification Rules: Set up rules to trigger notifications based on specific failure conditions (e.g., build failures, test failures).
Choose a Notification Service: Select whether Slack or email will be used as the notification medium.
Configure the Integration: Implement the integration between the CI/CD tool and the chosen notification service.
Test and Validate: Ensure that notifications are received correctly by the team.
Detailed Solution
1. Identify the CI/CD Tool

For this example, let's assume we’re using Jenkins for our CI/CD pipeline.

2. Define Notification Rules
Failures in build stages.
Specific test failures (e.g., unit tests failing).
Any deployment or release stage that fails.
3. Choose a Notification Service

In this case, Slack is chosen as the primary notification service due to its real-time communication capabilities and ease of integration with Jenkins.

4. Configure the Integration

We will use Jenkins’ built-in Slack plugin for notifications:

FLOWCHART · MERMAID
graph TD;
    A[Pipeline Starts] --> B1[Build Stage];
    B1 --> C1(Build Fails) OR D1(Build Succeeds);
    C1 --> E[Send Failure Message to Slack];
    D1 --> F[Continue Pipeline];
    C1 --> G[Test Stage];
    G --> H(Test Fails) OR I(Test Passes);
    H --> J[Send Test Failure Message to Slack];
    I --> K[Continue Pipeline];

Here’s a brief example of how the Jenkinsfile might look:

GROOVY
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def result = sh(script: 'make', returnStatus: true)
                    if (result != 0) {
                        slackSend color: '#FF0000', message: "Build failed for ${env.BUILD_NUMBER}"
                        error("Build failed")
                    }
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    def result = sh(script: 'make test', returnStatus: true)
                    if (result != 0) {
                        slackSend color: '#FF0000', message: "Tests failed for ${env.BUILD_NUMBER}"
                    }
                }
            }
        }
    }
}
5. Test and Validate
Run the pipeline manually to ensure that failure notifications are sent correctly.
Verify that the Slack messages contain relevant details like build number, stage name, and potential issues.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, setting up CI/CD pipeline failures requires identifying the tools, defining rules for when to notify, selecting a notification service, configuring the integration, and testing thoroughly. Common mistakes include not testing the setup, sending overly generic messages, or ignoring different severity levels.

#### Follow-up
questions might involve discussing:

Customizing the Message Content: What details should be included in the message?
Handling Different Team Members: Should notifications go to a specific channel or direct messaging?
Integration with Other Tools: Are there other tools (e.g., Jira) that need to be integrated for more comprehensive issue tracking?

**Comparison:**

```
Stage	Status
Build	Failed
Test	Passed
```

**Code:**

```
GROOVY
Copy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def result = sh(script: 'make', returnStatus: true)
                    if (result != 0) {
                        slackSend color: '#FF0000', message: "Build failed for ${env.BUILD_NUMBER}"
                        error("Build failed")
                    }
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    def result = sh(script: 'make test', returnStatus: true)
                    if (result != 0) {
                        slackSend color: '#FF0000', message: "Tests failed for ${env.BUILD_NUMBER}"
                    }
                }
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
- Jenkinsfile with stages defined as above
- Slack integration setup in Jenkins

Output:
- Slack message sent with failure details
- Jenkins pipeline halted at the build stage due to failure
```

**Data:**

```json
{
    "pipeline": {
        "stages": [
            {"name": "Build", "status": "failed"},
            {"name": "Test", "status": "passed"}
        ]
    },
    "slackMessage": {
        "color": "#FF0000",
        "text": "Build failed for ${env.BUILD_NUMBER}"
    }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Pipeline Starts] --> B1[Build Stage];
4. B1 --> C1(Build Fails) OR D1(Build Succeeds);
5. C1 --> E[Send Failure Message to Slack];
6. D1 --> F[Continue Pipeline];
7. C1 --> G[Test Stage];
8. G --> H(Test Fails) OR I(Test Passes);
9. H --> J[Send Test Failure Message to Slack];
10. I --> K[Continue Pipeline];

