---
type: "practice"
difficulty: "Deep"
---

# Explain why you'd pin third-party Actions in your GenAI CI to a full commit SHA instead of a tag like v4, what attack a mutable tag enables, and how you keep pinned SHAs updated without losing the guarantee.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ CI Cache Correctness](<../../../../TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure reliable and reproducible builds in GenAI CI pipelines, it's crucial to pin third-party Actions to specific commit SHAs rather than using mutable tags like v4. This practice prevents potential issues caused by updates or changes made after tagging, ensuring that your pipeline always uses the exact version of the Action intended at a particular point in time. Let’s break down why this is important and how we can manage these pinned SHAs effectively.

**Code:**

```python
# Example of pinning an Action using a commit SHA in a GitHub Actions workflow file
- name: Run Linter
  uses: action-name@SHA1234567890abcdef1234567890abcdef12345678 # Pin to this specific commit
```

**Example input / output:**

```text
# Example of a workflow file snippet with pinned Action using SHAs
on:
  push:
    branches: [ main ]
jobs:
  linting:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Run Linter
        uses: action-name@SHA1234567890abcdef1234567890abcdef12345678 # Pin to this specific commit
```

**Flow:**

1. Yes
2. No
3. GitHub Actions Workflow
4. Pin to SHA?
5. Pull Action at commit time
6. Use mutable tag, risk ofversion drift
7. Ensure reproducibility andpredictability in CI pipeline
8. Risk of unexpected changes

