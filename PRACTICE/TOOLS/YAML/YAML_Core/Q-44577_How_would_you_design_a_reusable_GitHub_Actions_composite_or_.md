---
type: "practice"
difficulty: "Deep"
---

# How would you design a reusable GitHub Actions composite or reusable workflow so downstream YAML stays thin, and what inputs/secrets propagation rules constrain the YAML interface?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Linting YAML](<../../../../TOOLS/YAML/YAML_Core/Linting_YAML.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this interview question effectively, we need to focus on designing a reusable GitHub Actions composite (or reusable workflow) that minimizes clutter in downstream workflows while ensuring flexibility and security through proper inputs and secrets propagation rules.

#### Triaging the Question

The core of the question revolves around creating a modular structure for GitHub Actions where you can abstract common tasks into reusable components. This not only keeps your primary YAML files lean but also improves maintainability and reusability across multiple projects or pipelines. The interviewer is probing your understanding of best practices in workflow design, input handling, and secrets management.

Designing Reusable Workflows

To create a reusable GitHub Actions composite, we need to break down the common tasks into smaller, manageable pieces that can be reused across different workflows. Here are the key steps:

Define Inputs: Inputs allow downstream workflows to customize the behavior of the composite workflow.
Propagate Secrets Securely: Ensuring that sensitive information is securely passed without exposing them in plain text.
Document and Version Control: Proper documentation and version control for reusable components.
Step-by-Step Solution

Define Inputs:

Use inputs to define parameters that the downstream workflow can pass during invocation.
YAML
# inputs.yaml
name: 'default'

Create a Reusable Workflow:

Create a separate YAML file for the reusable workflow with clear definitions of its behavior and expected inputs.
YAML
# .github/workflows/reusable-workflow.yml
name: Reusable Workflow

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    inputs:
      env_name:
        description: 'Environment Name'
        required: true
        default: 'default'

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'

      - name: Build and Test
        run: |
          echo "Building and testing for ${{ inputs.env_name }}"

Use the Reusable Workflow in Downstream Pipelines:

Reference the reusable workflow by its path, passing necessary inputs.
YAML
# .github/workflows/main-pipeline.yml
name: Main Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy-to-env:
    runs-on: ubuntu-latest
    needs: build-and-test

    steps:
      - name: Use Reusable Workflow
        uses: ./.github/workflows/reusable-workflow.yml
        with:
          env_name: ${{ github.ref }}

Securely Propagate Secrets:

Store sensitive information in GitHub Secret variables and pass them securely via the workflow's input mechanism.
YAML
# .github/workflows/main-pipeline.yml
name: Main Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy-to-env:
    runs-on: ubuntu-latest
    needs: build-and-test

    steps:
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'

      - name: Use Reusable Workflow
        env:
          SECRET_VAR: ${{ secrets.SECRET_VAR }}
        uses: ./.github/workflows/reusable-workflow.yml
        with:
          env_name: ${{ github.ref }}

#### Closing Summary
and Common Mistakes

By following these steps, you ensure that your downstream workflows remain clean and focused on their specific tasks while leveraging the power of reusable components. Key mistakes to avoid include:

Not documenting: Lack of clear documentation can lead to confusion.
Ignoring security: Failing to securely manage secrets can expose sensitive information.
Overcomplicating inputs: Too many or overly complex inputs can make workflows difficult to maintain.
Common Follow-Ups
How do you handle different environments?
What if the reusable workflow needs to call other actions?
How do you manage versioning of these reusable components?

**Code:**

```yaml
# inputs.yaml
name: 'default'

# .github/workflows/reusable-workflow.yml
name: Reusable Workflow

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    inputs:
      env_name:
        description: 'Environment Name'
        required: true
        default: 'default'

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'

      - name: Build and Test
        run: |
          echo "Building and testing for ${inputs.env_name}"

# .github/workflows/main-pipeline.yml
name: Main Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy-to-env:
    runs-on: ubuntu-latest
    needs: build-and-test

    steps:
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'

      - name: Use Reusable Workflow
        env:
          SECRET_VAR: ${{ secrets.SECRET_VAR }}
        uses: ./.github/workflows/reusable-workflow.yml
        with:
          env_name: ${{ github.ref }}
```

