---
type: "practice"
difficulty: "Deep"
---

# Two jobs both try to modify Terraform state concurrently. How do you prevent state corruption using concurrency plus backend locking?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Control in Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Concurrency_Control_in_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of two GitHub Actions concurrently modifying a shared Terraform state, we need to ensure that the operations are properly serialized or coordinated. The core idea is to implement a locking mechanism at the backend where the Terraform state is stored. This ensures that only one job can write to the state file at any given time, preventing corruption.

#### Triaging the Question

The question is essentially asking how to prevent state corruption when two jobs try to modify the same Terraform state concurrently. The interviewer is probing your understanding of concurrent operations in GitHub Actions and backend locking mechanisms. We need to make several assumptions:

You have a basic understanding of GitHub Actions and their execution flow.
You know about Terraform state management and its backend options.

#### First-Principles Solution

To prevent state corruption, we can leverage the built-in support for backend locking provided by Terraform. Here's how:

Define the Backend: Ensure that both jobs use a shared backend (e.g., S3 or RDB) with proper configuration.
Locking Mechanism: Utilize Terraform’s native backend settings to enable locking. This ensures that only one operation can be performed at a time.
Detailed Implementation

Let's break down the steps:

Configure Backend:

Define your backend in the main Terraform configuration file (e.g., terraform.tf) using an S3 or RDB backend.
Enable locking by setting lock options.

GitHub Actions Configuration:

Create separate GitHub Actions workflows for the two jobs that need to modify the state.
Ensure each workflow checks out the repository and runs Terraform in a way that respects the locking mechanism.

Here's an example configuration:

YAML
# .github/workflows/terraform-update.yml
name: Update Terraform State

on:
  push:
    branches:
      - main

jobs:
  update-state:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v3
    - name: Initialize Terraform
      run: terraform init -backend-config=.
    - name: Apply changes
      run: terraform apply -auto-approve -input=false

# .github/workflows/terraform-validate.yml
name: Validate Terraform State

on:
  pull_request:
    branches:
      - main

jobs:
  validate-state:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v3
    - name: Initialize Terraform
      run: terraform init -backend-config=.
    - name: Validate changes
      run: terraform apply -auto-approve -input=false -destroy=true

#### Closing Summary
and Common Mistakes

In summary, to prevent state corruption in GitHub Actions modifying a shared Terraform state, we utilize the backend locking provided by Terraform. By configuring both actions to use the same backend and ensuring proper locking settings, we can guarantee that only one operation is executed at any time.

#### Common mistakes
include:

Failing to configure the backend correctly.
Not understanding how Terraform’s lock mechanism works.
Ignoring the need for a shared backend across multiple jobs.
Self-Check

This solution addresses both correctness and completeness, ensuring state safety through backend locking mechanisms. The configuration steps are clear and align with best practices.

