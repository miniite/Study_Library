---
type: "practice"
difficulty: "Intermediate"
---

# How would you introduce CI/CD to a legacy codebase that currently has no automated tests?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Introducing CI CD to Legacy Systems](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Introducing_CI_CD_to_Legacy_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To introduce CI/CD into a legacy codebase without existing automated tests, we need to establish a systematic approach starting from minimal prerequisites, building up test coverage, and gradually integrating continuous integration and delivery practices. The key idea is to start with simple smoke tests and incrementally improve the testing strategy as more developers get on board.

The question here is really about assessing how you would introduce CI/CD into an environment that lacks any form of automated validation. This involves understanding the current state, setting realistic goals, choosing appropriate tools, and planning a phased implementation to minimize disruption while maximizing benefits. We'll assume the legacy codebase is in a common language like Python or Java, with a version control system (VCS) such as Git.

Let's break this down step by step:

Assess Current State: Identify the scope of the project and understand how developers currently work.
Set Up Version Control: Ensure all code changes are stored in a VCS like Git to facilitate tracking and collaboration.
Introduce Basic Testing: Start with basic unit tests using a framework like JUnit or pytest.
Implement Continuous Integration: Use CI tools such as Jenkins, GitLab CI/CD, or GitHub Actions.
Gradually Expand Test Coverage: As the codebase grows more testable, continue to add more comprehensive testing strategies.
Automate Deployment: Once tests are stable, automate deployment processes.

Here's a detailed step-by-step approach:

Assess Current State

Understand the project size and complexity.
Identify critical paths or components that must be developed first.

Set Up Version Control

Ensure all code is committed to Git.
Set up branch management strategies (e.g., Git Flow).

Introduce Basic Testing

Choose a testing framework appropriate for the language (JUnit, pytest).
Start with simple unit tests covering core functionalities.

Implement Continuous Integration

Select CI/CD tools based on existing infrastructure and developer preferences.
Set up initial CI pipelines to run basic tests after every commit.
Configure notifications to alert developers when tests fail.

Gradually Expand Test Coverage

Add integration tests to ensure components work together.
Implement end-to-end tests if necessary for complex systems.
Use coverage tools to measure test effectiveness and improve areas of low code coverage.

Automate Deployment

Once CI is stable, configure automated deployment pipelines.
Ensure security practices are in place (e.g., environment variable management).
Implement rollback mechanisms for failure-prone deployments.

**Comparison:**

```
Step	Action	Tools
1	Assess current state	Code review, stakeholder meetings
2	Set up version control	Git
3	Introduce basic testing	JUnit, pytest
4	Implement CI/CD	Jenkins, GitLab, GitHub Actions
5	Expand test coverage	PyTest, JUnit, integration tests
6	Automate deployment	Kustomize, Helm
```

**Code:**

```bash
# Example Jenkinsfile snippet
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def result = sh(script: 'mvn clean install', returnStatus: true)
                    assert result == 0, "Build failed"
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    def result = sh(script: 'kubectl apply -f deployment.yaml', returnStatus: true)
                    assert result == 0, "Deployment failed"
                }
            }
        }
    }
}
```

**Example input / output:**

```text
- **Input**: A legacy codebase with no automated tests.
- **Output**: CI/CD pipeline setup, initial unit tests running on every commit, and basic deployment automation.
```

**Data:**

```json
{
    "repo_url": "https://github.com/example/project",
    "branch_name": "main",
    "ci_tool": {
        "name": "Jenkins",
        "configuration_file": "/path/to/Jenkinsfile"
    },
    "test_coverage_targets": [
        {"component": "core", "target_percent": 80},
        {"component": "integrations", "target_percent": 60}
    ]
}
```

**Flow:**

1. Assess Current State
2. Setup Version Control
3. Introduce Basic Testing
4. Implement CI/CD
5. Expand Test Coverage
6. Automate Deployment

