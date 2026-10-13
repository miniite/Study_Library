---
type: "practice"
difficulty: "Deep"
---

# How do you implement a monorepo pipeline that only builds and deploys the services whose code actually changed?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing a monorepo pipeline with GitHub Actions, the goal is to optimize the build and deployment process by ensuring that only the services or components that have undergone changes are rebuilt and redeployed. This approach minimizes unnecessary work and reduces potential disruptions in the CI/CD pipeline. To achieve this, we need to carefully design our workflow configuration, leveraging Git’s branching strategy and custom actions.

In this scenario, you want to detect which services or modules have been modified and trigger only those specific builds and deployments. The core idea is to use a combination of branch filtering, git diff commands, and conditional logic within GitHub Actions workflows.

To start, let's assume our monorepo contains multiple microservices organized in subdirectories like service1, service2, etc. Each service has its own CI/CD configuration file or script that handles the build and deployment steps.

The key steps are:

Identify Changed Files: Use Git commands to identify which files have changed since the last commit.
Filter Services Based on Changes: Determine which services’ directories contain these changes.
Conditional Execution in GitHub Actions: Configure actions to only run if their respective service directories contain modified files.

We'll walk through each of these steps in detail.

**Code:**

```yaml
name: Monorepo CI/CD

on:
  push:
    branches:
      - main # or any other branch you're using

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [service1, service2] # List all services here

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Determine Changed Services
        id: changed_services
        run: |
          git diff --cached --name-only | grep -E "^(service[1-9])/.+" > .changed-services
          echo "::set-output name=changed::$(cat .changed-services)"

      - name: Filter and Build Specific Services
        if: steps.changed_services.outputs.changed != ''
        run: |
          for service in $(cat .changed-services); do
            if [[ ${{matrix.service}} == ${service%/.} ]]; then
              echo "Building ${{matrix.service}}"
              # Add your build commands here, e.g., ./build.sh
            fi
          done

      - name: Deploy Specific Services
        if: steps.changed_services.outputs.changed != ''
        run: |
          for service in $(cat .changed-services); do
            if [[ ${{matrix.service}} == ${service%/.} ]]; then
              echo "Deploying ${{matrix.service}}"
              # Add your deployment commands here, e.g., ./deploy.sh
            fi
          done
```

**Flow:**

1. Checkout Code
2. Determine Changed Services
3. Filter and Build SpecificServices
4. Filter and Deploy SpecificServices

