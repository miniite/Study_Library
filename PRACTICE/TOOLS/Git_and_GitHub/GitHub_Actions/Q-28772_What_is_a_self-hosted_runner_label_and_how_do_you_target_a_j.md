---
type: "practice"
difficulty: "Intermediate"
---

# What is a self-hosted runner label and how do you target a job to your own runner?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Self-hosted Runners](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Self-hosted_Runners.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
Self-hosted runners in GitHub Actions are devices that execute your workflows, and labels help identify which jobs should run on specific runners. In this explanation, we'll cover what self-hosted runners are, how to label them, and how to target specific jobs to these labeled runners.

#### Triage

Question: What is a self-hosted runner label in GitHub Actions, and how do you target a job to your own runner?
What's being probed: Understanding of GitHub Actions' architecture, the concept of labeling runners, and configuration capabilities.

#### Assumptions
You have basic knowledge of GitHub and its actions.

Step-by-step Solution:

Introduction to Self-Hosted Runners:

Self-hosted runners are machines you control that can be used as agents in your GitHub Actions workflows. They provide more flexibility compared to hosted runners, especially for running jobs on private networks or in environments with strict security policies.

Labeling Runners:

Labels allow you to categorize and identify specific self-hosted runners based on criteria like the operating system, architecture, or custom properties.
You can label a runner by adding label keys to its registration process using gh api. For example:
SH
gh api repositories/:owner/:repo/actions/runners/registration-token --method POST \
  -f "labels[]=linux" -f "labels[]=my-custom-label"

Targeting Jobs to Specific Runners:

To target a job to specific runners, you need to specify the label or labels that match your runner in the runs-on directive of your workflow.
For example:
YAML
jobs:
  build:
    runs-on: [self-hosted, linux, my-custom-label]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build
        run: make
This ensures that the job only runs on runners with the linux and my-custom-label labels.

Why Use Labels?

Labels help in managing a large number of self-hosted runners by organizing them into groups.
They enable you to prioritize certain jobs or ensure critical tasks run on specific machines, improving reliability and performance.

#### Common Mistakes

Forgetting to label your runners correctly can result in misrouting workflows.
Overlapping labels might cause unexpected behavior if multiple workflows target the same label.
Incorrectly specifying the runs-on directive might prevent certain jobs from running at all.

#### Closing Summary
and Follow-Ups:

Self-hosted runners with appropriate labels allow for granular control over job execution, ensuring that specific tasks run on the most suitable machines.
Regularly update your runner registration process to reflect changes in your environment or requirements.

