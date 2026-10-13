---
type: "practice"
difficulty: "Deep"
---

# What is a commit trailer like 'Signed-off-by', and why do some open-source infra projects require a DCO sign-off?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Signed Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Signed_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what a commit trailer like Signed-off-by is, we first need to explore how Git works under the hood and why certain practices are adopted in open-source projects, particularly infrastructural ones.

TL;DR

A commit trailer such as Signed-off-by is an additional piece of information that developers include in their Git commits. This practice ensures accountability for the code submitted by contributors, maintaining a high standard of quality and adherence to legal agreements. Some open-source infrastructure projects require a Developer Certificate of Origin (DCO) sign-off to formalize this commitment.

#### Triaging the Question

The question is probing your understanding of commit trailers in Git and their significance in open-source communities, particularly infrastructural ones. Key assumptions include:

You are familiar with basic Git operations.
You understand the importance of contributor agreements in large-scale software projects.
You can explain how these practices support legal and community standards.

#### First-Principles Solution
What is a Commit Trailer?

In Git, commit trailers provide additional metadata to your commits beyond just the message body. A common trailer is Signed-off-by, which is used by some open-source projects to ensure that contributors are legally bound to the terms of the project's contributor agreement.

Why Use Signed-off-by?
Legal Binding: The DCO (Developer Certificate of Origin) is a widely adopted standard for maintaining the provenance and legal license compliance of contributions in open-source projects, especially those hosted on GitHub.
Accountability: By signing off, contributors explicitly state that they have committed to the terms outlined by the project's contributor agreement, thereby ensuring accountability.
How it Works

When you commit changes, a Signed-off-by trailer might look like this:

PLAINTEXT
Signed-off-by: John Doe <john.doe@example.com>

This tells everyone involved in the review process that John Doe is accountable for his contribution and has read and agreed to the DCO.

#### Example
Workflow
Developer makes changes.
Developer commits with a commit message and Signed-off-by trailer.
The commit is pushed to a Git repository, e.g., GitHub.
Pull requests are reviewed by maintainers who ensure that all Signed-off-by trailers match the contributor's name and email.
Why Do Some Open-Source Projects Require DCO Sign-offs?
Legal Compliance: Many open-source projects operate under specific licenses (e.g., MIT, Apache) that require contributors to agree to certain terms.
Code Quality Assurance: Ensuring that all contributions are from people who have read the code of conduct and agreed to its terms helps maintain a high standard of quality and ethical behavior.

#### Common Mistakes
Forgetting to add Signed-off-by in your commit trailer can lead to delays or rejections during pull request reviews.
Misusing the Signed-off-by field (e.g., using someone else's email) can result in legal disputes and harm community trust.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, a Signed-off-by commit trailer is an essential practice for maintaining accountability and legal compliance in open-source projects. While it adds a bit of extra work during the commit process, it significantly enhances the overall quality and trustworthiness of contributions.

#### Common mistakes
include:

Forgetting to include or incorrectly formatting the Signed-off-by trailer.
Not understanding the DCO and its implications for project contributors.

For follow-ups, consider how different projects handle similar practices (e.g., CLA-assistant for GitHub repositories) and discuss potential alternatives that balance simplicity with thoroughness.

