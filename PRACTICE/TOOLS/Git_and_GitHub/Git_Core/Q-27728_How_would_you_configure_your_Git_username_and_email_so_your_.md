---
type: "practice"
difficulty: "Basic"
---

# How would you configure your Git username and email so your schema commits on the team repo are correctly attributed to you?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Signed Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Signed_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your Git commits are correctly attributed to you when working with a team repository, you need to set up your global Git username and email. This step is crucial as it ensures that every commit you make gets tagged with the correct identity, making collaboration easier and ensuring proper attribution for code ownership.

First, let's break down what we're dealing with: configuring your Git settings involves setting up a few pieces of information globally so they are applied to all repositories on your local machine. Here’s how you can do it:

Set Your Global Username: This is the name that will appear in your commit messages.
Set Your Global Email Address: This should be associated with your GitHub account, as it ties directly to your identity.

Let's go through these steps one by one:

Step 1: Set Up Your Global Username and Email

You can configure this using Git’s command line interface. Open a terminal (or your preferred command-line tool) on your computer and run the following commands:

SH
git config --global user.name "Your Name"
git config --global user.email you@example.com

Replace "Your Name" with your actual name, and you@example.com with your GitHub email address. This will set these values globally for all Git projects on your machine.

Step 2: Verify the Configuration

To make sure that your settings have been applied correctly, run:

SH
git config --list

This command lists out all the global configurations, including user.name and user.email, allowing you to verify that they match what you just set up.

Full Example Workflow

Here’s a full example of how this might look in practice:

Open your terminal.
Set your username:
SH
git config --global user.name "Alice Smith"
Set your email address:
SH
git config --global user.email alice.smith@example.com
Verify the settings:
SH
git config --list

#### Closing Summary
and Common Mistakes

Setting up Git correctly is straightforward, but there are a few common mistakes to avoid:

Mismatched Email: Ensure that your email address in Git matches the one associated with your GitHub account.
Multiple Emails: If you have multiple emails (e.g., work and personal), make sure to set an email specifically for Git and GitHub.
Overwriting Settings: Be cautious when setting global settings, as they apply across all repositories. You can always override these settings on a per-repository basis if needed.

By following these steps, your commits will be correctly attributed to you in the team repository, ensuring clear authorship of code changes.

