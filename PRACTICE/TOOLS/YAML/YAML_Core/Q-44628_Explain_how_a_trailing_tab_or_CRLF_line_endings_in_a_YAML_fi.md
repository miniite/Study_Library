---
type: "practice"
difficulty: "Intermediate"
---

# Explain how a trailing tab or CRLF line endings in a YAML file cause cross-platform parse failures and how you enforce LF and no-tabs in CI.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Indentation Pitfalls](<../../../../TOOLS/YAML/YAML_Core/Indentation_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with YAML, especially across different operating systems, it's crucial to ensure consistent line endings (CR/LF/Line Feed) and avoid using tabs. A trailing tab or CRLF line endings can cause cross-platform parse failures because YAML parsers handle these differently depending on the system. This issue often arises in CI pipelines where the environment might not be configured properly to handle such inconsistencies.

#### Triage

The core of the question is about understanding how YAML parsing works across different operating systems and ensuring consistent formatting standards through CI. Specifically, we need to address two issues: enforcing LF line endings (commonly used on Unix-based systems) and eliminating trailing tabs. The interviewer might also be probing your knowledge of CI/CD pipelines, configuration management tools like Jenkins or GitLab, and best practices for file validation.

#### First-Principles Solution
Understand YAML Parsing Behavior: YAML parsers interpret CRLF as the end of a line on Windows but not on Unix-based systems. Similarly, trailing tabs can cause syntax errors because they are treated differently by various editors and parsing libraries.
Enforce LF Line Endings: Use Git hooks or pre-commit scripts to convert all files to use LF line endings before committing them to the repository.
Eliminate Trailing Tabs: Utilize tools like sed or regular expressions in a script to remove trailing tabs from YAML files.
Step-by-Step Solution

Install Necessary Tools:

Ensure you have Git and any required scripting languages (e.g., Bash, Python) installed on your CI servers.

Configure Git Hooks for LF Line Endings:

BASH
# Install the git hooks from the standard distribution
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit

# Edit the pre-commit hook to include a check for CRLF and trailing tabs
sed -i 's/^#!/#/'; echo "#!/bin/bash" > .git/hooks/pre-commit
cat <<EOF >> .git/hooks/pre-commit
find . -name "*.yaml" -exec dos2unix {} \;
find . -name "*.yml" -exec dos2unix {} \;
EOF
chmod +x .git/hooks/pre-commit

Write a Script to Remove Trailing Tabs:

BASH
# Create a script to clean up YAML files
cat > cleanup_yaml.sh <<EOF
#!/bin/bash
find . -name "*.yaml" -exec sed -i 's/[ \t]*$//' {} +
find . -name "*.yml" -exec sed -i 's/[ \t]*$//' {} +
EOF
chmod +x cleanup_yaml.sh

Integrate Scripts into CI Pipeline:

YAML
# Example Jenkinsfile snippet
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Run the cleanup script as part of the build process
                sh './cleanup_yaml.sh'
            }
        }
    }

    post {
        always {
            // Ensure file endings are consistent before committing
            sh 'git add .'
            sh 'git commit -m "Cleanup YAML files and ensure LF line endings"'
            sh 'git push origin main'
        }
    }
}

Test Locally Before Pushing:

Run the cleanup_yaml.sh script locally to ensure it works as expected.
Manually test a few YAML files with CRLF and trailing tabs.

#### Closing Summary

By ensuring consistent line endings and eliminating trailing tabs, we can prevent cross-platform parse failures in YAML files. This is crucial for maintaining compatibility across different CI environments and avoiding errors during pipeline execution.

#### Common Mistakes
and Follow-Ups
Incorrect Line Ending Conversion: Failing to properly convert CRLF to LF can lead to syntax errors.
Missing Trailing Tab Removal: Overlooking the removal of trailing tabs might result in silent failures during parsing.
Ignoring CI/CD Pipeline Integration: Not integrating these checks into the CI pipeline means the issues will persist, leading to intermittent failures.

**Code:**

```bash
# Pre-commit Git hook for LF line endings
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit

cat <<EOF >> .git/hooks/pre-commit
#!/bin/bash
find . -name "*.yaml" -exec dos2unix {} \;
find . -name "*.yml" -exec dos2unix {} \;
EOF
chmod +x .git/hooks/pre-commit

# Cleanup script for YAML files
cat > cleanup_yaml.sh <<EOF
#!/bin/bash
find . -name "*.yaml" -exec sed -i 's/[ \t]*$//' {} +
find . -name "*.yml" -exec sed -i 's/[ \t]*$//' {} +
EOF
chmod +x cleanup_yaml.sh

# Example Jenkinsfile snippet
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh './cleanup_yaml.sh'
            }
        }
    }

    post {
        always {
            sh 'git add .'
            sh 'git commit -m "Cleanup YAML files and ensure LF line endings"'
            sh 'git push origin main'
        }
    }
}
```

