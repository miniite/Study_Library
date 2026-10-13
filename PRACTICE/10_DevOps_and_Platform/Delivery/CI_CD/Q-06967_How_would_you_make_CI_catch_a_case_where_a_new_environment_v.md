---
type: "practice"
difficulty: "Intermediate"
---

# How would you make CI catch a case where a new environment variable is required by the code but never added to the deploy config, before it fails at runtime in prod?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Pre-merge vs Post-merge Failures](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pre-merge_vs_Post-merge_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your Continuous Integration (CI) pipeline catches cases where a new environment variable is needed by the code but not added to the deployment configuration, we can implement a series of checks both in the code and during the CI process. The goal is to detect this issue early so that it doesn't cause failures at runtime in production.

#### Triage
Understanding the Problem

The core question here involves ensuring that all necessary environment variables are defined before deploying new changes. This primarily requires static analysis, possibly complemented by dynamic checks if feasible. Static analysis can be performed during code review or using tools, while dynamic checks can be run as part of the CI pipeline.

#### Assumptions
The application uses environment variables for configuration.
Environment variables are stored in a central repository (like Git).
There is a clear definition of required and optional environment variables per environment.
CI/CD tools are available and configurable.

#### First-Principles Solution
1. Static Analysis via Code Review and Linting

We can use static code analysis tools to flag missing or unused environment variables. Tools like ESLint for JavaScript, PyLint for Python, etc., have plugins that can scan the source code for references to environment variables.

#### Example
Using ESLint with a Custom Rule

JAVASCRIPT
// .eslintrc.json
{
  "rules": {
    "no-undefined-env-var": function(context) {

#### cons
t envVars = context.getEnvironment("env").getEnvironmentVariables();

      return function(node) {
        if (node.type === 'Identifier' && envVars.indexOf(node.name) === -1) {
          return new ErrorNode(node, "Environment variable not defined");
        }
        return null;
      };
    }
  }
}
2. Integration with CI Pipeline

We can integrate the static analysis tool into our CI pipeline to ensure that code changes are reviewed before deployment.

#### Example
Jenkins Pipeline

GROOVY
pipeline {
  agent any
  stages {
    stage('Static Analysis') {
      steps {
        script {
          def result = sh(script: 'npm run lint', returnStatus: true)
          if (result != 0) {
            error("Linting failed. Please fix the issues.")
          }
        }
      }
    }
    // Other stages like Build, Test, etc.
  }
}
3. Runtime Checks

While static analysis is essential, it might not catch all cases where a new environment variable could be required in the future. Therefore, we can implement dynamic checks to verify that all expected environment variables are present before starting the application.

#### Example
Python Application with Env Var Check

PYTHON
import os

required_env_vars = {'API_KEY', 'DB_HOST'}

def check_env_vars():
    missing_vars = required_env_vars - set(os.environ.keys())
    if missing_vars:
        raise ValueError(f"Missing environment variables: {missing_vars}")

check_env_vars()

#### Closing Summary
and Common Mistakes

In summary, the solution involves a combination of static code analysis to catch immediate issues during development, integration with CI pipelines to enforce this practice, and runtime checks to ensure no missing variables cause crashes in production.

#### Common Mistakes/Follow-Ups

Over-reliance on Static Analysis: While useful, it doesn't always catch all cases.
Neglecting Runtime Checks: Ensuring that the application can fail gracefully without crashing is crucial.
Manual Over-Dependence: Not automating checks leads to human error and delay.

By addressing these points, we ensure a robust CI/CD process that minimizes runtime issues in production.

**Comparison:**

```
Static Check	Dynamic Check
ESLint	Environment Variable Checks in Code
Jenkins	Runtime Checks in Application
```

**Code:**

```

// ESLint custom rule example { "rules": { "no-undefined-env-var": function(context) { const envVars = context.getEnvironment("env").getEnvironmentVariables();

CODE
Copy
  return function(node) {
    if (node.type === 'Identifier' && envVars.indexOf(node.name) === -1) {
      return new ErrorNode(node, "Environment variable not defined");
    }
    return null;
  };
}

} }

CODE
Copy

```groovy
// Jenkins Pipeline example
pipeline {
  agent any
  stages {
    stage('Static Analysis') {
      steps {
        script {
          def result = sh(script: 'npm run lint', returnStatus: true)
          if (result != 0) {
            error("Linting failed. Please fix the issues.")
          }
        }
      }
    }
    // Other stages like Build, Test, etc.
  }
}
PYTHON
Copy
# Python application check example
import os

required_env_vars = {'API_KEY', 'DB_HOST'}

def check_env_vars():
    missing_vars = required_env_vars - set(os.environ.keys())
    if missing_vars:
        raise ValueError(f"Missing environment variables: {missing_vars}")

check_env_vars()
```

**Example input / output:**

```text
Input: npm run lint, Jenkins pipeline execution

Output: Linting failed. Please fix the issues.
```

**Data:**

```json
{
  "required_env_vars": ["API_KEY", "DB_HOST"]
}
```

**Flow:**

1. Code Analysis
2. Static Analysis Tool
3. CI Pipeline Integration
4. Runtime Check in Application

