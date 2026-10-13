---
type: "practice"
difficulty: "Basic"
---

# How would you clone a starter observability-as-code template and immediately point origin at your own new GitHub repository?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Sharing Projects on GitHub](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sharing_Projects_on_GitHub.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clone a starter observability-as-code template from GitHub and then change its origin to point to a newly created repository, you need to follow a few straightforward steps. This process involves using Git commands interactively in your terminal or command line interface (CLI).

Firstly, the question is asking us to perform two main tasks:

Clone an existing repository.
Change the remote origin URL of that cloned repository.

Key assumptions here are:

You have Git installed on your machine and set up with a GitHub account.
The starter observability-as-code template is available via a public or shared private repository link on GitHub.

Let's break down the steps:

Clone the Starter Template:

Open your terminal/CLI.
Use the git clone command followed by the URL of the observability-as-code template repository. For example:
SH
git clone https://github.com/org-name/template-repo.git
This will create a local copy (a new directory) on your machine named after the repository.

Navigate to Cloned Repository Directory:

Change into the newly cloned directory using cd command.
SH
cd template-repo

Change Remote Origin URL:

Use the git remote set-url command to change the origin to point to your new repository. You'll need the URL of your own GitHub repository, which might look something like this:
SH
git remote set-url origin https://github.com/your-github-username/new-template-repo.git

Push Changes (Optional):

If you want to push any changes or initial configuration files from the starter template to your new repository, use git add ., git commit -m "Initial commit", and then git push.
SH
git add .
git commit -m "Initial commit"
git push origin master  # Assuming you want to push to the main branch

Here's a quick summary of what we did:

Cloned an existing repository using Git.
Changed the remote origin URL to point to our new GitHub repository.
Optionally, pushed initial changes.

#### Common mistakes
to avoid include not confirming the correct remote origin after setting it or forgetting to commit and push any necessary files before pushing them to your new repository.

