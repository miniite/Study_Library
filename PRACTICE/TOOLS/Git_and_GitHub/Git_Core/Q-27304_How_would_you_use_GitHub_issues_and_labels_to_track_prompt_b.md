---
type: "practice"
difficulty: "Basic"
---

# How would you use GitHub issues and labels to track prompt bugs and hallucination reports in a small LLM project?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Secure Workflows for Forks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Secure_Workflows_for_Forks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

GitHub issues and labels are powerful tools for managing and tracking defects, like bugs or hallucinations, in an AI model’s development lifecycle. In this scenario, we can leverage GitHub issues to create a structured system for bug reports and hallucination detection, and use labels to categorize and prioritize these reports.

Firstly, let's understand what we mean by "prompt bugs" and "hallucinations." Prompt bugs occur when the input prompt produces unexpected or incorrect output. Hallucinations refer to outputs that contain information not present in the input data but are generated due to model misinterpretation or errors.

To effectively manage these issues, we can follow a step-by-step process:

Create an Issue Template: Define a template for reporting both bugs and hallucinations.
Use Labels Wisely: Apply appropriate labels to each issue based on the type of defect and its severity.
Automate Tracking and Notifications: Integrate GitHub workflows to automate notifications and updates.

Now, let's dive into how we can implement this system from first principles:

Step 1: Define an Issue Template

Start by creating a template for reporting issues. This template should include fields such as:

Title: Brief description of the issue.
Type: Whether it is a bug or hallucination.
Severity: High, Medium, Low (to prioritize).
Reproducibility Steps: Detailed steps to reproduce the issue.
Expected Output: What the output should be ideally.
Actual Output: The output received.

Here's an example of what the template might look like:

MARKDOWN
# [Type: Bug/Hallucination] - [Severity: High/Medium/Low]
## Title
Describe the issue briefly here.

## Type
Is this a bug or hallucination?

## Severity
Assign severity based on impact and urgency (High, Medium, Low).

## Reproducibility Steps
Provide detailed steps to reproduce the issue.

## Expected Output
What should the model output ideally?

## Actual Output
What did the model actually output?
Step 2: Use Labels for Categorization and Prioritization

Define a set of labels that can be used to categorize and prioritize issues. Some common labels might include:

bug: For bugs in the model.
hallucination: For hallucinations.
reproducible: To mark issues where steps are clear.
low-severity, medium-severity, high-severity: To prioritize fixes based on impact.
Step 3: Automate Tracking and Notifications

GitHub Actions can be used to automate the tracking process. You can set up workflows that automatically update issue statuses or send notifications when an issue is created, updated, or resolved. Here’s a simple example of how you might configure a GitHub Action:

YAML
name: Bug Hallucination Tracker

on:
  issues:
    types: [opened, edited]

jobs:
  notify-bug-hallucination:
    runs-on: ubuntu-latest
    steps:
      - name: Notify via Slack/Email
        run: |
          if [[ "${{ github.event.issue.labels.name }}" == "bug" || "{{ github.event.issue.labels.name }}" == "hallucination" ]]; then
            echo "New bug or hallucination reported. Check the issue details."
          fi

#### Closing Summary
and Common Mistakes/Follow-Ups

Using GitHub issues and labels effectively can significantly enhance the quality of an LLM project by ensuring that all bugs and hallucinations are systematically tracked and addressed. Key mistakes to avoid include not using consistent labeling practices, neglecting to document reproducibility steps, or failing to prioritize issues based on their severity.

#### Follow-ups
might include:

Regularly reviewing the status of reported issues.
Improving the template for better clarity.
Training team members on proper issue reporting and label usage.

