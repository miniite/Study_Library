---
type: "practice"
difficulty: "Basic"
---

# In a GitHub Actions workflow YAML file what does the 'on' key control?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Indentation Pitfalls](<../../../../TOOLS/YAML/YAML_Core/Indentation_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The on key in a GitHub Actions workflow YAML file controls when and under what circumstances the workflow should be triggered. Essentially, it acts as the "trigger" or "event listener" that decides which actions to take based on specific events.

To break down the question:

What is being asked: The on key defines the conditions or events that will cause the workflow to run.
Key points to consider:
Commonly used triggers include branch creation, push events, pull request events, etc.
Understanding these triggers helps you design workflows that respond appropriately to various actions in your repository.

Now let's go through a step-by-step explanation from first principles:

#### Overview
of GitHub Actions Workflow Structure: A typical workflow YAML file looks like this:

YAML
name: My Workflow

on:
  # Specify the trigger here

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build
        run: make

Purpose of the on Key: The on key specifies which events or conditions will trigger the workflow to start. Common examples include:

push: Triggers on every push event.
pull_request: Triggered when a pull request is opened, closed, or updated.
schedule: Runs the workflow at specific times.
paths: Runs based on file paths.

#### Example
of Different on Configurations:

For a simple push trigger, you would use:
YAML
name: CI

on:
  push:
    branches: [ "main" ]
For a pull request check:
YAML
name: PR Check

on:
  pull_request:
    branches: [ "main", "develop" ]

#### Closing Summary
and Common Mistakes:

A common mistake is to forget to specify the on key, which means the workflow will not run at all.
Another frequent issue is using overly broad triggers (like push: { branches: ["*"] }) which can lead to unnecessary runs.

By understanding and correctly configuring the on key, you ensure that your workflows are triggered only when needed, leading to efficient use of resources and timely responses to code changes.

