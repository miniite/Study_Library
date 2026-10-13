---
type: "practice"
difficulty: "Basic"
---

# How would you use a GitHub template repository to standardize the folder structure of every new LLM demo your team builds?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Sharing Projects on GitHub](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sharing_Projects_on_GitHub.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To use a GitHub template repository for standardizing the folder structure in every new LLM demo, we'll leverage Git's templating capabilities and GitHub Actions to automate the process of setting up a consistent project layout. This approach ensures that all team members follow the same structure, reducing friction and enhancing code quality.

The key idea is to create a template repository on GitHub that contains the desired folder structure and configuration files. When a new demo starts, we can initialize this template in a new local repository or directly clone it with the template applied.

#### Triage
What's Being Probed
Understanding of Git Templates: How familiar are you with Git's templating feature?
GitHub Actions Understanding: Do you know how to use GitHub Actions for automation?
Version Control Best Practices: Are you aware of best practices in version control that involve templates?

#### Assumptions
We have a team working on multiple LLM demos.
The folder structure and configuration files are standardized across all demos.

#### First-Principles Solution

Create the Template Repository:

On GitHub, create a new repository dedicated to your template.
Structure this repository with the desired folder hierarchy and relevant configuration files (e.g., .gitignore, README.md, etc.).

Enable Git Templating:

Enable "Template" repositories on GitHub by turning off the option to clone an empty repo as a project.
This ensures that when someone forks or clones your template, they get all the necessary files.

Set Up a Custom Template in GitHub Settings:

Go to Settings -> Repository -> Templates.
Click on "Enable repository template" and select your template repository.

Using Git Templates in Local Projects:

When starting a new project, use the command:
SH
git clone --template=<TEMPLATE_REPO_URL> <NEW_PROJECT_DIR>
This will initialize the new project with all the necessary files from your template repository.

GitHub Actions for Automation (Optional):

Create a GitHub Action that runs when a new project is initialized.
Use this action to automate steps like setting up development environments, installing dependencies, or performing initial setup tasks.

#### Example
Workflow

Team Member Starts a New Project:

SH
git clone --template=https://github.com/team/template.git my_new_llm_demo

Initial Setup by GitHub Action (if configured):

The GitHub Action will run, setting up the environment and performing initial configurations.

#### Closing Summary

Using a GitHub template repository standardizes the folder structure for every new LLM demo your team builds, ensuring consistency across projects. This approach leverages Git's templating feature and can be enhanced with automation via GitHub Actions to streamline the setup process further.

#### Common Mistakes/Follow-Ups
Not Enabling Template Repository: Failing to enable template repositories on GitHub.
Ignoring Configuration Files: Forgetting to include necessary configuration files in the template.
Manual Setup Steps: Relying too much on manual steps rather than automating with GitHub Actions.

**Code:**

```
SH
Copy
# Example command to initialize a new project from a Git template
git clone --template=https://github.com/team/template.git my_new_llm_demo
```

**Flow:**

1. Yes
2. No
3. Initiate New Project
4. Use Git Template?
5. Clone with Template
6. Run GitHub Actions (Optional)
7. Manual Setup Steps

