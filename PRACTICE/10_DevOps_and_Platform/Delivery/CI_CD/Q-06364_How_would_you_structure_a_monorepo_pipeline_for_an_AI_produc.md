---
type: "practice"
difficulty: "Deep"
---

# How would you structure a monorepo pipeline for an AI product where a change to the prompt package, the retriever package, or the model config each triggers only the relevant evals?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shared Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a CI/CD pipeline for an AI product with a monorepo structure, we need to ensure that changes in specific packages (such as prompt, retriever, or model config) trigger only relevant evaluations, optimizing both efficiency and resource usage. This approach involves leveraging Git hooks, Docker containers, and strategic Jenkinsfile configurations.

#### Triaging the Question

The core of this question is about creating a pipeline that can selectively run tests based on which package has been modified. We need to identify which components are affected by the changes (e.g., prompt, retriever) and only execute relevant evaluation pipelines for those specific packages. The interviewer is probing your understanding of Git workflows, Docker containerization, and Jenkins configuration.

#### Assumptions
Monorepo Structure: A single repository containing multiple projects or packages.
CI/CD Tools: Jenkins for CI and CD processes.
Git Hooks: Used to run custom scripts before or after specific events (e.g., commit).
Docker Containers: To encapsulate the environment in which tests are executed.

#### First-Principles Solution
Identify Affected Packages: Detect changes made to prompt, retriever, and model config packages.
Selective Evaluation Pipelines: Run only relevant evaluation pipelines based on detected changes.
Environment Configuration: Use Docker containers to ensure consistent environments for tests.

#### Implementation
Git Hooks:
Set up pre-commit hooks in the repository to automatically run a script that checks which package has been modified.
Jenkinsfile Configuration:
Create multiple Jenkins jobs, each responsible for a specific package (e.g., prompt-eval, retriever-eval, and model-config-eval).
Use conditional logic in the Jenkinsfile to execute only the relevant evaluation job based on the detected changes.
Docker Containers:
Create Docker images tailored for each environment required by different packages.

#### Example
Jenkinsfile
GROOVY
pipeline {
    agent any

    stages {
        stage('Check Changes') {
            steps {
                script {
                    def modifiedFiles = sh(script: 'git diff --cached --name-only', returnStdout: true).trim().split('\n')
                    def promptModified = modifiedFiles.any { it.contains('prompt')}
                    def retrieverModified = modifiedFiles.any { it.contains('retriever')}
                    def configModified = modifiedFiles.any { it.contains('model config')}
                }
            }
        }

        stage('Run Relevant Evaluations') {
            when {
                expression { promptModified }
            }
            steps {
                script {
                    container('prompt-env') {
                        sh 'make test-prompt'
                    }
                }
            }
        }

        stage('Run Relevant Evaluations') {
            when {
                expression { retrieverModified }
            }
            steps {
                script {
                    container('retriever-env') {
                        sh 'make test-retriever'
                    }
                }
            }
        }

        stage('Run Relevant Evaluations') {
            when {
                expression { configModified }
            }
            steps {
                script {
                    container('config-env') {
                        sh 'make test-config'
                    }
                }
            }
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

This setup ensures that only relevant evaluation pipelines are triggered, saving time and resources. Common mistakes include not properly configuring the Git hooks or Jenkinsfile logic, leading to unnecessary evaluations. Follow-ups might involve discussing how to handle edge cases (e.g., multiple modifications), integrating with other tools like SonarQube for code quality checks, and scaling the solution for larger repositories.

**Comparison:**

| Stage         | Condition              | Action                                      |
|---------------|------------------------|---------------------------------------------|
| Check Changes | Any modified file      | Detect which files are changed             |
| Run Relevant  | promptModified         | Run `make test-prompt` in `prompt-env`      |
| Run Relevant  | retrieverModified      | Run `make test-retriever` in `retriever-env`|
| Run Relevant  | configModified         | Run `make test-config` in `config-env`      |

**Code:**

```
GROOVY
Copy
pipeline {
    agent any

    stages {
        stage('Check Changes') {
            steps {
                script {
                    def modifiedFiles = sh(script: 'git diff --cached --name-only', returnStdout: true).trim().split('\n')
                    def promptModified = modifiedFiles.any { it.contains('prompt')}
                    def retrieverModified = modifiedFiles.any { it.contains('retriever')}
                    def configModified = modifiedFiles.any { it.contains('model config')}
                }
            }
        }

        stage('Run Relevant Evaluations') {
            when {
                expression { promptModified }
            }
            steps {
                script {
                    container('prompt-env') {
                        sh 'make test-prompt'
                    }
                }
            }
        }

        stage('Run Relevant Evaluations') {
            when {
                expression { retrieverModified }
            }
            steps {
                script {
                    container('retriever-env') {
                        sh 'make test-retriever'
                    }
                }
            }
        }

        stage('Run Relevant Evaluations') {
            when {
                expression { configModified }
            }
            steps {
                script {
                    container('config-env') {
                        sh 'make test-config'
                    }
                }
            }
        }
    }
}
```

**Example input / output:**

```

Given input:

git diff --cached --name-only output:
PLAINTEXT
Copy
prompt/utils.py
retriever/data_loader.py
model_config/inference.yml

Output (Jenkinsfile execution):

Check Changes stage identifies changes in all three packages.
Run Relevant Evaluations stages:
prompt-env container runs make test-prompt
retriever-env container runs make test-retriever
config-env container runs make test-config
```

**Data:**

```json
{
  "modifiedFiles": [
    "prompt/utils.py",
    "retriever/data_loader.py",
    "model_config/inference.yml"
  ]
}
```

**Flow:**

1. Check Changes
2. promptModified
3. Run make test-prompt inprompt-env
4. retrieverModified
5. Run make test-retriever inretriever-env
6. configModified
7. Run make test-config inconfig-env

