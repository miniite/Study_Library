---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up a pipeline that only deploys when a git tag matching a pattern is pushed?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pipeline Triggers and Path Filters](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_Triggers_and_Path_Filters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a CI/CD pipeline that only deploys when a specific git tag is pushed, we need to configure both our Continuous Integration (CI) and Continuous Deployment (CD) processes. The key idea here is to ensure that the deployment step in our CD process runs conditionally based on the tags being pushed.

#### Triage

The question asks for setting up a CI/CD pipeline with a conditional deployment mechanism. This involves:

Identifying Tags: Determine the pattern or name of the git tag.
CI Trigger: Configure the CI system to trigger only when specific tags are pushed.
Conditional Deployment: Modify the CD step so that it only proceeds if the correct tag is detected.

#### First-Principles Solution

Let’s walk through this step-by-step:

Choose Your CI/CD Tools: Common choices include Jenkins, GitLab CI, GitHub Actions, and CircleCI.
Identify Tag Pattern: Decide on a specific pattern for your tags (e.g., v1.*, beta-*).
Configure Git Repository: Ensure the repository is properly set up to accept and use tags.
Set Up CI Trigger:
In Jenkins, this might involve setting up a webhook or polling strategy based on tag names.
For GitHub Actions, you can configure the .github/workflows file to only trigger on certain events (like push with specific tags).
Conditional Deployment in CD Pipeline: Modify the deployment step so that it checks for the correct tag before proceeding.

#### Example
with Jenkins

Here’s a simplified example of how this might look in a Jenkins pipeline:

GROOVY
pipeline {
    agent any

    environment {
        TAG_PATTERN = 'v[0-9]+\\.[0-9]+'
    }

    triggers {
        // Poll SCM for push events that match the tag pattern
        pollSCM('H * * * *', { params.TAG ==~ /${env.TAG_PATTERN}/ })
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building application...'
                // Build logic here
            }
        }

        stage('Deploy') {
            when {
                expression { return params.TAG =~ /${env.TAG_PATTERN}/ }
            }
            steps {
                echo "Deploying version ${params.TAG}..."
                // Deployment logic here
            }
        }
    }
}

#### Example
with GitHub Actions

For GitHub Actions, you can use the on keyword to specify when the workflow should run:

YAML
name: Deploy on Tag

on:
  push:
    tags:
      - 'v*.*'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Build and deploy
      if: startsWith(github.ref, 'refs/tags/v')
      run: |
        echo "Deploying version $(github.event.tag.name)..."
        # Deployment logic here

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, to set up a CI/CD pipeline that deploys only when specific tags are pushed:

Identify the Tag Pattern: Define what your tags look like (e.g., v1.0).
Configure the CI System: Use triggers or events in tools like Jenkins or GitHub Actions.
Conditional Deployment Logic: Ensure the deployment step runs only if the correct tag is detected.

#### Common mistakes
include:

Failing to properly define the tag pattern, leading to unnecessary deployments.
Not configuring the CI/CD system correctly, resulting in missed deploys or accidental deployments.

#### Follow-ups
could involve discussing additional security measures, handling rollback scenarios, and ensuring that non-tagged pushes do not trigger unintended builds.

**Code:**

```

pipeline { agent any

CODE
Copy
environment {
    TAG_PATTERN = 'v[0-9]+\\.[0-9]+'
}

triggers {
    pollSCM('H * * * *', { params.TAG ==~ /${env.TAG_PATTERN}/ })
}

stages {
    stage('Build') {
        steps {
            echo 'Building application...'
        }
    }

    stage('Deploy') {
        when {
            expression { return params.TAG =~ /${env.TAG_PATTERN}/ }
        }
        steps {
            echo "Deploying version ${params.TAG}..."
        }
    }
}

}

CODE
Copy

```yaml
name: Deploy on Tag

on:
  push:
    tags:
      - 'v*.*'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Build and deploy
      if: startsWith(github.ref, 'refs/tags/v')
      run: |
        echo "Deploying version $(github.event.tag.name)..."
```

**Example input / output:**

```text
No specific input-output examples are necessary here as the configurations are based on patterns and conditions rather than explicit inputs/outputs.
```

**Flow:**

1. Yes
2. No
3. CI Pipeline Trigger
4. Is Tag Matching?
5. Build
6. Skip Deployment
7. Deploy to Production

