---
type: "practice"
difficulty: "Intermediate"
---

# How do you store a secret like an API key so it is not visible in .gitlab-ci.yml?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Protected Environments](<../../../../TOOLS/GitLab/GitLab_CI/Protected_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely manage secrets such as API keys in your .gitlab-ci.yml file, without exposing them directly, GitLab provides a robust system called Environment Variables and Secret Variables. This approach ensures that sensitive information is stored outside of your repository, making it inaccessible to anyone who clones the project.

First, let's clarify what the question is asking and what GitLab expects us to do:

Objective: Store an API key securely in your CI/CD pipeline.
Probing Points: Understanding how to avoid hardcoding sensitive information directly into .gitlab-ci.yml, which can lead to security risks if the repository is publicly accessible.
Step-by-Step Solution
1. Using Environment Variables

Environment variables are a common way to store secrets in GitLab CI/CD pipelines. These can be set at various levels: project, group, or instance-wide. You can access these variables using ${VARIABLE_NAME} syntax within your .gitlab-ci.yml file.

Setup Steps:

Go to the Settings > CI/CD section of your GitLab repository.
Under the Variables tab, you can add a new variable named API_KEY.
Set the value for this variable as needed. You can also mark it as "Masked", which will display asterisks (*) in place of the actual value.
2. Using Secret Variables

For more sensitive information, GitLab provides Secret Variables. These are stored as encrypted values and cannot be viewed or modified directly by developers. To use a secret variable:

Setup Steps:

Go to Settings > CI/CD > Variables.
Add a new variable named API_KEY_SECRET.
Set the value of this variable, but mark it as a "Secret Variable".
Now you can access this in your .gitlab-ci.yml file using ${!API_KEY_SECRET}.
3. Using .gitignore and External Configuration Files

Another approach is to use an external configuration file that is ignored by Git (e.g., .env). This way, the API key is not stored directly in the repository but can still be accessed during CI/CD runs.

Setup Steps:

Create a .env file containing your API key.
Add this file to your .gitignore so it is never committed to the repository.
Use an dotenv or similar package to load environment variables from .env in your application. For GitLab CI, you can mount this file during job execution.

#### Example
of a .gitlab-ci.yml File Using Secret Variables
YAML
stages:
  - deploy

deploy-job:
  stage: deploy
  script:
    - echo "Deploying with API_KEY_SECRET=${!API_KEY_SECRET}"

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, using GitLab's Environment or Secret Variables is a secure way to manage secrets like API keys. Always ensure that sensitive information is not stored directly in your repository and consider using external configuration files for additional security layers.

#### Common mistakes
include:

Hardcoding sensitive data into .gitlab-ci.yml or other source code.
Not marking variables as "Masked" or "Secret Variables", leading to accidental exposure of secrets.
Ignoring the importance of using .gitignore to protect sensitive files.

If you have any follow-ups, such as how to manage different environments (development, staging, production), let’s discuss that next!

**Code:**

```yaml
stages:
  - deploy

deploy-job:
  stage: deploy
  script:
    - echo "Deploying with API_KEY_SECRET=${!API_KEY_SECRET}"
```

