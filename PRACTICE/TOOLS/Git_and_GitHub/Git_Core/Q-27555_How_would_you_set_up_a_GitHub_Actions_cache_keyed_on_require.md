---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up a GitHub Actions cache keyed on requirements.txt so your telemetry-analysis test workflow installs dependencies faster?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CI Cache Correctness](<../../../../TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a GitHub Actions cache keyed on requirements.txt to speed up dependency installation in a telemetry-analysis test workflow, we need to follow these steps:

#### Overview
We'll configure a GitHub Actions job that caches the environment based on changes in requirements.txt. This will avoid reinstalling dependencies when only tests have changed.

#### Triage
The question is asking how to efficiently manage dependency installation by leveraging caching mechanisms provided by GitHub Actions. The core idea is to ensure that if requirements.txt hasn't changed, we don’t re-install the dependencies, which can save significant time in our build process.

#### First Principles Solution

Create a GitHub Actions workflow.
Use the cache action to store and restore dependencies based on changes in requirements.txt.
Customize the workflow to run tests only when necessary.

#### Closing Summary
This setup ensures that dependency installation is faster by caching the environment, thus improving the overall efficiency of our test pipeline.

#### Common Mistakes/Follow-Ups

Not understanding how cache keys work.
Incorrectly configuring the cache action.
Overlooking the need to clean up cached dependencies if requirements.txt changes significantly.

Now let's dive into the implementation details:

YAML
# .github/workflows/install-dependencies.yml

name: Install Dependencies

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  cache-dependencies:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      # Cache dependencies based on the state of requirements.txt
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'

      - name: Install Dependencies
        run: |
          pip install --upgrade pip
          pip install -r requirements.txt

      # Cache dependencies for faster installation on subsequent runs
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-

  run-tests:
    runs-on: ubuntu-latest
    needs: cache-dependencies

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Install Dependencies (from cache)
        if: needs.cache-dependencies.outputs.cache-hit != 'true'
        run: |
          pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run Tests
        run: |
          python -m unittest discover

