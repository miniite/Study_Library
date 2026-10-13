---
type: "practice"
difficulty: "Deep"
---

# Your dbt slim-CI build relies on a production manifest.json artifact and it went stale after a hotfix ran outside CI; how would you keep the deferred state manifest trustworthy so the diff isn't wrong?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data and Analytics CI Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the manifest.json file remains trustworthy in your dbt slim-CI setup, especially after a hotfix that ran outside of CI, we need to focus on maintaining an accurate and up-to-date deferred state manifest. This involves understanding how the manifest is generated, identifying the source of truth for model dependencies, and implementing robust mechanisms to sync changes between environments.

Firstly, let's clarify what the manifest.json file does. It contains metadata about all the dbt models in a project, including their dependencies, generated tables, and other relevant information. When this manifest goes stale, it leads to incorrect diff reports, which can result in unnecessary rework or missed issues during hotfixes.

To keep the deferred state manifest trustworthy, we need to ensure that:

The manifest.json is updated with the latest schema changes.
Dependencies between models are correctly reflected.
Any new models added post-hotfix are included in the build process.

Now let's break down the steps from first principles:

Step 1: Identify Source of Truth for Model Dependencies

Ensure that all model dependencies are explicitly defined within your dbt models or project files. This could be through explicit depends_on blocks, or by referencing a central configuration file.

dbt_project.yml
models/
model_a.dbt
model_b.dbt
Step 2: Implement CI-Driven Manifest Updates

To keep the manifest.json accurate, we need to ensure that every time a model is updated or added, it triggers an update of the manifest. This can be achieved by configuring your build process to run a dbt command that generates the manifest after each model change.

BASH
# Example dbt command in CI/CD pipeline
dbt run-operation generate_manifest --vars "output_path: 'path/to/manifest.json'"
Step 3: Use Centralized Version Control for Manifest Updates

Ensure that all changes to models and their dependencies are committed to version control. This way, the manifest.json is always in sync with the latest codebase.

BASH
# Example Git commit message
git add . && git commit -m "Add new model and update manifest"
Step 4: Synchronize Changes Between Environments

When a hotfix runs outside of CI, it can introduce changes that are not yet reflected in your main branch. To handle this:

Tag the Hotfix: Ensure the hotfix is tagged and pushed to all relevant environments.
Sync Manifests: Manually or via scripts, ensure that the manifest.json from the hotfix environment is copied over to your CI environment.
BASH
# Example script to sync manifest files
cp path/to/hotfix/manifest.json path/to/ci_environment/manifest.json
Step 5: Monitor and Alert on Manifest Staleness

Implement monitoring tools that alert you when the manifest.json is stale or inconsistent. Tools like dbt's manifest_diff command can be used to compare the manifest between environments.

BASH
# Example dbt command for diffing manifests
dbt run-operation diff_manifests --vars "project_path: 'path/to/your/project'"
Step 6: Automate the Build Process

Ensure that your CI pipeline is automated and runs on every push to the main branch. This ensures that the manifest.json is always up-to-date with the latest changes.

YAML
# Example GitHub Actions workflow
name: dbt Build

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.8'
    - name: Install dependencies
      run: |
        pip install dbt
    - name: Generate manifest
      run: |
        dbt run-operation generate_manifest --vars "output_path: 'path/to/manifest.json'"

#### Closing Summary
and Common Mistakes/Follow-Ups

By following these steps, you can maintain a trustworthy manifest.json that accurately reflects the state of your dbt project. Common mistakes include not committing changes to version control, failing to update dependencies explicitly, or not having robust monitoring in place.

For follow-ups, consider how different environments (e.g., development, staging, production) are managed, and whether additional tools like semantic layers could further enhance the accuracy of your manifest updates.

**Comparison:**

```
Model	Depends On
model_a	model_b
model_b
Manifest File Path	Status
path/to/manifest.json	Up-to-date
path/to/hotfix/manifest.json	Stale
```

**Code:**

```yaml
# Example GitHub Actions workflow for dbt CI/CD
name: dbt Build

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.8'
    - name: Install dependencies
      run: |
        pip install dbt
    - name: Generate manifest
      run: |
        dbt run-operation generate_manifest --vars "output_path: 'path/to/manifest.json'"
```

**Example input / output:**

```text
# Example input and output for diffing manifests
Input:
- Project A manifest in main branch
- Hotfix branch manifest

Output:
- Diff report showing discrepancies between the two manifests
```

**Data:**

```json
{
  "models": [
    {
      "name": "model_a",
      "depends_on": ["model_b"]
    },
    {
      "name": "model_b"
    }
  ]
}
```

**Flow:**

1. Hotfix
2. Tag and Push to Main
3. Sync Manifest Files
4. Run CI Pipeline
5. Generate and UpdateManifests

