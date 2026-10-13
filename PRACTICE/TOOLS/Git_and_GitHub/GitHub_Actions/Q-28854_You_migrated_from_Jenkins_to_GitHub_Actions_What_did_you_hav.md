---
type: "practice"
difficulty: "Deep"
---

# You migrated from Jenkins to GitHub Actions. What did you have to redesign around shared libraries, credentials, and agents?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Secrets in Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Secrets_in_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating from Jenkins to GitHub Actions, one of the most critical aspects is understanding how these components are managed differently in each system. In Jenkins, shared libraries, credentials, and agent management can be complex due to its extensive plugin ecosystem and custom configuration options. With GitHub Actions, these areas are streamlined but require a different approach to achieve similar functionality.

#### Triaging the Question

The primary focus here is on redesigning components such as shared libraries, credentials, and agents in the context of moving from Jenkins to GitHub Actions. The core issue is understanding how these elements can be managed more effectively within GitHub Actions while maintaining their intended functionalities.

#### Assumptions
Shared Libraries: Jenkins uses Groovy scripts for managing reusable code snippets, whereas GitHub Actions relies on actions and jobs.
Credentials Management: Both systems handle secrets differently, with Jenkins often using plugins like Vault or AWS Secrets Manager, and GitHub Actions employing environment variables or the built-in secret management.
Agents: In Jenkins, agents are managed by a master/slave architecture, whereas in GitHub Actions, jobs can be orchestrated within the same machine.
Step-by-Step Solution
Shared Libraries

In Jenkins, shared libraries allow for reusing code and configuration across multiple pipelines. In contrast, GitHub Actions uses workflows and actions to achieve similar goals.

Library Structure:

Jenkins: Typically stored in a Git repository with Groovy scripts.
GitHub Actions: Use the actions keyword to define reusable parts within your workflows.

Usage:

Jenkins: Call libraries using Jenkins functions or plugins.
GitHub Actions: Use uses step to reference actions (equivalent of shared libraries).
YAML
# Example GitHub Action
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
Credentials Management

Handling secrets and credentials is crucial for security.

Jenkins:

Use Jenkins plugins like Vault or AWS Secrets Manager to manage secrets.
Store sensitive information in the Jenkins credential store.

GitHub Actions:

Utilize environment variables for storing secrets.
Securely set these using GitHub's secret management feature.
YAML
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
Agents

Agent management differs significantly between Jenkins and GitHub Actions.

Jenkins:

Managed by a master/slave architecture, where agents are provisioned and managed centrally.
Use labels to manage agent allocation based on environment or OS type.

GitHub Actions:

Jobs run directly in the same machine as GitHub Actions runner.
No need for separate agent management; jobs can be orchestrated across different runners with various configurations.
YAML
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Run a one-line script
        run: echo Hello, world!

#### Closing Summary

Migrating from Jenkins to GitHub Actions requires adapting shared libraries, credential management, and agent configurations. By leveraging GitHub’s native features for these components, you can maintain the functionality of your CI/CD processes while benefiting from better integration and security.

#### Common Mistakes/Follow-Ups
Not Properly Securing Secrets: Mismanaging secrets by hardcoding them in workflows or not using environment variables.
Overlooking Environment Labeling: Not correctly configuring job steps to run on the appropriate agent labels, leading to misconfigured jobs.
Misusing Actions vs. Jobs: Confusing actions with jobs and failing to structure workflows effectively.

**Code:**

```yaml
# Example GitHub Action
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
```

**Data:**

```yaml
# Jenkins Shared Library Example
@Library('my-shared-library') _

pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        echo 'Building'
      }
    }
  }
}
```

**Flow:**

1. Shared Libraries
2. Define Actions
3. Credentials Management
4. Set Environment Variables
5. Agents
6. Run Jobs On Same Machine

