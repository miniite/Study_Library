---
type: "practice"
difficulty: "Basic"
---

# How do you access the value of an environment variable inside a run step on a Linux runner?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Self-hosted Runners](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Self-hosted_Runners.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To access the value of an environment variable inside a run step on a Linux runner in GitHub Actions, you need to use the $VAR syntax, where VAR is the name of your environment variable. This involves setting up the environment variables in your workflow and then referencing them correctly within your job's steps.

#### Triage

The question asks about accessing an environment variable inside a run step on a Linux runner using GitHub Actions. The interviewer might be probing your understanding of how to set up and reference environment variables in workflows, as well as familiarity with basic syntax used in YAML configuration files for specifying these variables.

#### First-Principles Solution
Setting Up Environment Variables: You can define environment variables at the workflow or job level using the env keyword.
Accessing Environment Variables: In a run step, you can access an environment variable by prefixing it with $.

Here's a basic example of how to set up and use environment variables in GitHub Actions:

YAML
name: Sample Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    env:
      MY_VAR: "Hello, World!"

    steps:
    - name: Print the value of MY_VAR
      run: echo $MY_VAR

#### Closing Summary
and Common Mistakes/Follow-Ups

In this example, MY_VAR is defined as an environment variable at the job level using the env keyword. Inside a run step, you can access it by prefixing $ before its name. If the environment variable isn't set correctly or if the syntax is incorrect, the step will fail.

#### Common mistakes
include:

Forgetting to use the $ prefix.
Not defining the environment variable in the workflow YAML file.
Using a variable name that doesn't match what was defined.

#### Follow-ups
could involve discussing how to handle secrets (using secrets keyword instead of env), or talking about best practices for managing sensitive information in workflows.

**Code:**

```yaml
name: Sample Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    env:
      MY_VAR: "Hello, World!"

    steps:
    - name: Print the value of MY_VAR
      run: echo $MY_VAR
```

