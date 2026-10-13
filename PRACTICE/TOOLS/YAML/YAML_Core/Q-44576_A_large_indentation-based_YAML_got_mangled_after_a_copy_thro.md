---
type: "practice"
difficulty: "Intermediate"
---

# A large indentation-based YAML got mangled after a copy through a chat tool converting spaces. How do you defensively detect non-breaking spaces or tabs in CI before they reach kubectl?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Indentation Pitfalls](<../../../../TOOLS/YAML/YAML_Core/Indentation_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of detecting non-breaking spaces (NBSP) or tabs in a YAML file before it reaches kubectl, we need to ensure that our Continuous Integration (CI) pipeline performs thorough validation checks on the YAML files. This is crucial because incorrect indentation can cause Kubernetes resources to fail deployment, leading to downtime and potential data loss.

#### Triage

The core question here revolves around identifying non-breaking spaces or tabs in a YAML file before it's applied using kubectl. The interviewer is probing your understanding of CI/CD pipelines, the importance of file integrity checks, and your ability to implement robust validation mechanisms. We will assume that the chat tool converting spaces introduces errors like inserting NBSP instead of standard spaces.

#### First-Principles Solution
Check for Non-Breaking Spaces (NBSP): Use regular expressions or a library to identify and replace NBSP with normal spaces.
Validate YAML Syntax: Ensure the file is syntactically correct before applying it to Kubernetes resources.
CI/CD Pipeline Integration: Integrate these checks into your CI pipeline so that any invalid YAML files are caught early.
Step-by-Step Solution

Identify and Replace NBSP:

We can use Python's re module to find and replace NBSP with normal spaces.

Validate YAML Syntax:

Use a library like PyYAML for parsing the file and catching any syntax errors.

Integrate into CI/CD Pipeline:

Add steps in your Jenkinsfile or GitHub Actions workflow to run these checks before deployment.
Code Example
PYTHON
import re
import yaml

def clean_yaml(file_path):
    with open(file_path, 'r') as file:
        content = file.read()

    # Replace NBSP with normal spaces
    content = re.sub(u'\xa0', u' ', content)

    try:
        # Parse the cleaned YAML content
        parsed_yaml = yaml.safe_load(content)
        return True, parsed_yaml
    except yaml.YAMLError as e:
        print(f"YAML error: {e}")
        return False, None

# Example usage in a CI/CD step
clean_yaml_result = clean_yaml('path/to/your/yaml/file.yaml')
if not clean_yaml_result[0]:
    # Handle the error appropriately (e.g., fail the build)
    raise ValueError("Invalid YAML detected")
Integrating into Jenkinsfile

Here’s how you might integrate this check into a Jenkinsfile:

GROOVY
pipeline {
    agent any
    stages {
        stage('Lint YAML') {
            steps {
                sh 'python3 path/to/clean_yaml.py'
            }
        }
        // Other stages...
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to defensively detect non-breaking spaces or tabs in a YAML file before it reaches kubectl, we need to implement checks in our CI/CD pipeline. By replacing NBSP with normal spaces and validating the YAML syntax, we can ensure that only valid and properly formatted files are applied. Common mistakes include not integrating these checks early enough in your workflow or failing to handle errors gracefully.

For follow-ups, you might discuss how to further automate this process using static code analysis tools, or explore more advanced techniques like linting with specific Kubernetes YAML linters.

**Code:**

```python
import re
import yaml

def clean_yaml(file_path):
    with open(file_path, 'r') as file:
        content = file.read()

    # Replace NBSP with normal spaces
    content = re.sub(u'\xa0', u' ', content)

    try:
        # Parse the cleaned YAML content
        parsed_yaml = yaml.safe_load(content)
        return True, parsed_yaml
    except yaml.YAMLError as e:
        print(f"YAML error: {e}")
        return False, None

# Example usage in a CI/CD step
clean_yaml_result = clean_yaml('path/to/your/yaml/file.yaml')
if not clean_yaml_result[0]:
    # Handle the error appropriately (e.g., fail the build)
    raise ValueError("Invalid YAML detected")
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. No
6. Start
7. Read YAML file
8. Check for NBSP
9. Clean content by replacingNBSP with spaces
10. Precleaned content valid?
11. Lint or parse YAML to detectsyntax errors
12. YAML invalid?
13. Fail the build
14. Continue pipeline
15. File contains invalid characters,fail the build

