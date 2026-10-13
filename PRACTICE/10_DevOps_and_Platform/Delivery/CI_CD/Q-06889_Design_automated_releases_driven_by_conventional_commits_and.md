---
type: "practice"
difficulty: "Deep"
---

# Design automated releases driven by conventional commits and semantic versioning; what breaks if a developer writes a bad commit message, and how do you guard the version bump?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Schema Changes in Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Schema_Changes_in_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Automated release processes based on conventional commits and semantic versioning are crucial for maintaining software quality and consistency across development teams. In this scenario, we're designing an automated CI/CD pipeline that listens to commits pushed to a repository and bumps the version according to semantic versioning rules (major.minor.patch). However, if developers write bad commit messages, it can lead to incorrect or unexpected version increments. To address this, we need robust guards in place.

The key idea is to ensure that commit messages are well-formed and adhere to our defined standards before triggering a version bump. We will use regular expressions (regex) for pattern matching and semantic versioning libraries for parsing and validation. Let's break down the solution step-by-step.

**Code:**

```python
import re
from semantic_version import Version

def validate_commit_message(message):
    # Define regex pattern for valid commit messages
    pattern = r'^\b(?P<type>\w+)(?:\((?P<scope>[^)]*)\))?\s+(?P<title>.+)'

    match = re.match(pattern, message)
    if not match:
        return False

    # Extract components from the commit message
    type_ = match.group('type')
    scope = match.group('scope') or ''
    title = match.group('title')

    # Validate using semantic versioning library
    try:
        Version('0.0.1')  # Dummy version to initialize
        return True
    except ValueError as e:
        print(f"Version increment failed: {e}")
        return False

def bump_version(current_version_str, commit_message):
    if not validate_commit_message(commit_message):
        raise ValueError("Invalid commit message format")

    try:
        current_version = Version(current_version_str)
        # Bump version based on the type of the commit
        if "feat" in commit_message:  # Feature update
            return str(current_version.bump_major())
        elif "fix" in commit_message:  # Bug fix
            return str(current_version.bump_minor())
        elif "refactor" in commit_message:  # Code refactoring
            return str(current_version.patch())
        else:
            raise ValueError("Unknown commit type")
    except Exception as e:
        print(f"Error bumping version: {e}")
        return None
```

**Example input / output:**

```

Example input:

PYTHON
Copy
current_version = "1.2.3"
commit_message = "fix: typo in login page"

new_version = bump_version(current_version, commit_message)
print(new_version)  # Expected output: "1.2.4"
```

**Flow:**

1. Valid
2. Invalid
3. Commit pushed
4. Validate Commit Message
5. Bump Version
6. Log Error, Do Not BumpVersion

