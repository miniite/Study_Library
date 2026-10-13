---
type: "practice"
difficulty: "Intermediate"
---

# How would you architect a frontend deploy so a bad release can be rolled back in seconds by a non-engineer during an incident?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architected correctly, deploying and rolling back a frontend application should be straightforward—so even a non-engineer can manage it quickly during an incident. To achieve this, we need to focus on decoupling our deployment processes from complex dependencies and ensuring that rollback mechanisms are robust, automated, and accessible.

In essence, the question is about designing a deploy pipeline with two main components:

Automated Deployment: Ensuring the application can be deployed quickly without manual intervention.
Rollback Mechanism: Creating a reliable way to revert changes if something goes wrong.

Let's break down each component from first principles:

Step-by-Step Solution
1. Automated Deployment

We'll use an automated CI/CD pipeline, where any code change triggers a deployment process without human interaction. For this example, let’s assume we're using GitHub as the version control system and Jenkins for continuous integration.

Key Components:
Source Control Repository: Github (or similar)
Build Pipeline: Jenkins
Containerization: Docker or Podman
2. Rollback Mechanism

To enable a non-engineer to roll back in seconds, we need to ensure that the rollback process is as simple and intuitive as possible. This involves:

Maintaining multiple versions of the application.
Using a versioned deployment strategy.
Key Components:
Environment Setup: Different environments (staging, production)
Versioning Strategy: Label each deployment with a unique identifier
Rollback Plan: A clear rollback procedure that can be followed by anyone
Detailed Implementation
1. Continuous Integration and Deployment Pipeline

We start by setting up a Jenkins pipeline to build, test, and deploy the application automatically on every commit.

GROOVY
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    echo 'Building the application...'
                    sh 'docker build -t myapp:latest .'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Running tests...'
                    sh 'docker run --rm myapp:latest npm test'
                }
            }
        }

        stage('Deploy') {
            when { branch 'main' } // Only deploy from main branch
            steps {
                script {
                    echo 'Deploying the application...'
                    sh "kubectl apply -f deployment.yaml"
                }
            }
        }
    }
}
2. Versioned Deployment Strategy

We ensure that each deployment is tagged and can be rolled back to a specific version if needed.

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myapp:$(TAG)

In the above YAML, TAG is a placeholder that would be replaced with an actual version number during deployment.

3. Rollback Procedure

To facilitate rollback by non-engineers, we create a simple command-line utility or a script that can take a specific tag as input and revert to that version.

BASH
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: $(basename $0) <version>"
    exit 1
fi

TAG=$1
kubectl rollout undo deployment/myapp-deployment --to-revision=$TAG
echo "Rolled back to version $TAG"

#### Closing Summary
and Common Mistakes
Summary: By using a CI/CD pipeline, maintaining versioned deployments, and creating an intuitive rollback script, we ensure that even non-engineers can quickly roll back in case of issues.

#### Common Mistakes
Overcomplicating the deployment process with unnecessary tools or steps.
Not adequately testing rollbacks before actual incidents occur.
Failing to document the rollback procedure clearly.

**Comparison:**

```
Deployment Command	Action
docker build -t myapp:latest .	Build Docker image
kubectl apply -f deployment.yaml	Apply deployment YAML file
./rollback.sh <version>	Rollback to specific version
```

**Code:**

```

pipeline { agent any

CODE
Copy
stages {
    stage('Build') {
        steps {
            script {
                echo 'Building the application...'
                sh 'docker build -t myapp:latest .'
            }
        }
    }

    stage('Test') {
        steps {
            script {
                echo 'Running tests...'
                sh 'docker run --rm myapp:latest npm test'
            }
        }
    }

    stage('Deploy') {
        when { branch 'main' } // Only deploy from main branch
        steps {
            script {
                echo 'Deploying the application...'
                sh "kubectl apply -f deployment.yaml"
            }
        }
    }
}

}

CODE
Copy

```bash
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: $(basename $0) <version>"
    exit 1
fi

TAG=$1
kubectl rollout undo deployment/myapp-deployment --to-revision=$TAG
echo "Rolled back to version $TAG"
```

**Example input / output:**

```

Deployment command output:

CODE
Copy

```plaintext
Building the application...
Running tests...
Deploying the application...
Rolled back to version v1.2.3
```

**Data:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myapp:$(TAG)
```

**Flow:**

1. GitHub Commit
2. CI Pipeline Trigger
3. Caching and Building DockerImage
4. Jenkins Deploy Stage
5. Kubernetes Deployment Apply
6. Health Check (Optional)
7. Rollback Script Available
8. Non-Engineer Runs RollbackScript

