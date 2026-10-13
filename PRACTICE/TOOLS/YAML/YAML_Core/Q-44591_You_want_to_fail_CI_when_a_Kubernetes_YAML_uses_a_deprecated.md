---
type: "practice"
difficulty: "Intermediate"
---

# You want to fail CI when a Kubernetes YAML uses a deprecated API. What tools (kubeconform, pluto) do you wire in and how do they read the YAML kind/apiVersion to decide?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kustomize Patching](<../../../../TOOLS/YAML/YAML_Core/Kustomize_Patching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a CI pipeline fails when a Kubernetes YAML file uses a deprecated API, we can leverage tools like kubeconform and pluto. These tools help us validate the YAML files against the Kubernetes schema to detect issues such as using outdated APIs. Here’s a step-by-step approach on how to integrate these tools into your CI/CD pipeline:

Firstly, kubeconform is a command-line tool that validates Kubernetes manifests against the official API schemas hosted at https://api.k8s.io. It checks for both deprecated and non-compliant resources in YAML or JSON files. On the other hand, pluto is another validation tool that helps with schema enforcement.

To integrate these tools into your CI pipeline:

Install kubeconform: This can be done by adding a step to install kubeconform as part of your CI/CD setup.
Write a CI script: Modify or create a Jenkinsfile, GitHub Action, or another CI tool configuration file to include steps that run kubeconform.
Parse the YAML files: Use the tools to validate each Kubernetes manifest by parsing the kind/apiVersion fields.

The key idea here is to make sure that your YAML files are validated against the latest Kubernetes schema and that any use of deprecated APIs triggers a build failure in CI.

**Code:**

```bash
# Example Jenkinsfile snippet to integrate kubeconform

pipeline {
    agent any
    stages {
        stage('Validate YAML') {
            steps {
                script {
                    def yamlFiles = findYamlFiles() // Custom function to locate YAML files
                    for (yamlFile in yamlFiles) {
                        sh "kubeconform --out=json ${yamlFile} | jq '.[].errors[]' > validation_output.json"
                        def errors = readJSON file: 'validation_output.json'
                        if (!errors.empty) {
                            error "Kubernetes manifest validation failed: ${errors}"
                        }
                    }
                }
            }
        }
    }
}
```

**Flow:**

1. Start CI
2. Find YAML files
3. Run `kubeconform` on eachfile
4. Parse JSON output for errors
5. If errors, fail build; otherwisecontinue
6. End CI

