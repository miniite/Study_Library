---
type: "practice"
difficulty: "Basic"
---

# How would you use a GitHub pull request to get review on a new PromQL recording rule you added for a dashboard?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pull Requests](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pull_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively manage changes and ensure quality in monitoring configurations, using a GitHub pull request (PR) is essential. In this scenario, you're adding a new PromQL recording rule to a dashboard. Here's how you would use GitHub PRs to get review:

TL;DR: You create a branch for your feature, commit the new PromQL rule, open a PR, and engage your team for feedback.

#### Triage

The question is asking about using GitHub pull requests as part of a code review process. Specifically, it’s probing on how to integrate version control practices with monitoring configurations, ensuring that changes are reviewed before being merged into the main branch. Assumptions include having access to a Git repository and familiarity with PromQL and Grafana dashboards.

#### First-Principles Solution

Clone the Repository: Ensure you have a local copy of the existing repository.

BASH
git clone https://github.com/example/team-monitoring.git
cd team-monitoring

Create a Feature Branch:

This branch will be specific to your new PromQL rule, making it easier to track changes.
BASH
git checkout -b add-new-recording-rule

Add the New Recording Rule:

Open or create the file where recording rules are defined (usually in a .prom file).
Add your new PromQL rule, ensuring it aligns with existing best practices.

Commit and Push Changes:

Commit your changes with clear messages.
BASH
git commit -am "Add new recording rule for dashboard"
git push origin add-new-recording-rule

Open a Pull Request (PR):

Go to the GitHub web interface, navigate to the branch add-new-recording-rule, and click on the 'Compare & pull request' button.
Fill out the PR details: title should clearly describe what you've done, and in the description, add links to any relevant documentation or discussions.

Engage Your Team:

Assign reviewers who are familiar with PromQL and dashboards.
Engage your team for a thorough review of the new rule, discussing its implications on monitoring.

Iterate Based on Feedback:

Address any comments or concerns from reviewers.
Make necessary adjustments and recommit changes if needed.

Merge Once Approved:

Once approved, merge the PR into the main branch after addressing all feedback.

#### Closing Summary

Using GitHub pull requests for reviewing new PromQL recording rules is a best practice in AIOps to ensure that monitoring configurations are accurate, reliable, and have been reviewed by peers. This process helps catch potential issues early and ensures consistency across dashboards.

#### Common Mistakes/Follow-Ups
Not Creating a Separate Branch: Always use separate branches for new features or changes to keep the main branch clean.
Lack of Descriptive Commit Messages: Clear commit messages are crucial for understanding changes without needing to review every line.
Ignoring Feedback: Failing to address feedback can lead to unnecessary rework and delays.

**Code:**

```bash
# Clone repository
git clone https://github.com/example/team-monitoring.git

# Change into the repo directory
cd team-monitoring

# Create a new feature branch
git checkout -b add-new-recording-rule

# Add new recording rule (example .prom file)
echo "new_recording_rule = sum by (job) (rate(http_requests_total[5m]))" >> rules/prometheus_rules.prom

# Commit and push changes
git commit -am "Add new recording rule for dashboard"
git push origin add-new-recording-rule
```

**Example input / output:**

```

N/A, but a typical input-output would be:

PLAINTEXT
Copy
Input (New PromQL rule):
"new_recording_rule = sum by (job) (rate(http_requests_total[5m]))"

Output (Commit Message):
"Add new recording rule for dashboard"
```

