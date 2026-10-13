---
type: "practice"
difficulty: "Deep"
---

# How would you structure a monorepo pipeline so a change to a shared feature library re-tests exactly the models that consume it and nothing else?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shared Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To structure a monorepo pipeline effectively, ensuring that changes to a shared feature library only re-test the specific models that consume it is crucial for maintaining efficiency and reducing test noise. This involves setting up targeted tests and leveraging CI/CD tools to isolate and prioritize the relevant tests.

The key steps are:

Identify Dependencies: Determine which models in each project depend on the shared feature library.
Isolate Tests: Create separate test suites or groups for different projects that use the shared library.
Conditional Builds and Tests: Configure your CI/CD to run only specific tests based on the changed files.

**Comparison:**

```
Project	Dependent Models	Test Suite
ProjectA	Model1, Model2	TestSuiteA
ProjectB	None	TestSuiteB
```

**Code:**

```python
# Example of a Jenkinsfile snippet for isolating builds

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def featureLibraryChanged = params.featureLibraryCommits // hypothetical parameter
                    if (featureLibraryChanged) {
                        echo "Running specific tests for dependent projects"
                        sh 'make test-specific'
                    } else {
                        echo "Building and testing all projects as usual"
                        sh 'make build-and-test-all'
                    }
                }
            }
        }
    }
}
```

**Example input / output:**

```text
Input: A change is detected in the shared feature library.
Expected Output:
1. If only a specific model in ProjectA depends on this library, then only the tests for ProjectA are run.
2. If multiple models in different projects depend on the changed library, those specific test suites are executed.

Example dry-run:
Input (change in `shared-feature-library`):
- Dependency check: ProjectA: Yes, ProjectB: No
Output: Run tests for ProjectA only.
```

**Data:**

```yaml
# Example YAML configuration for a GitLab CI/CD pipeline

stages:
  - test
  - deploy

test-specific-project1:
  stage: test
  script:
    - make tests-for-feature-library-dependent-projects

test-all:
  stage: test
  script:
    - make full-tests
```

**Flow:**

1. Yes
2. No
3. Identify Changes in FeatureLibrary
4. Is Change in a DependentModel?
5. Test Specific Models
6. Run Full Pipeline Tests
7. Rerun Only Relevant Tests forDependencies

