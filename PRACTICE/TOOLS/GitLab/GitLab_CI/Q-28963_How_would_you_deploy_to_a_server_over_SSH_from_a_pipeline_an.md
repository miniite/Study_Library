---
type: "practice"
difficulty: "Intermediate"
---

# How would you deploy to a server over SSH from a pipeline and store the SSH key safely?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Protected Environments](<../../../../TOOLS/GitLab/GitLab_CI/Protected_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deploy to a server over SSH from a GitLab CI pipeline, we need to ensure that our SSH key is stored securely while still allowing us to authenticate with the target server. Here’s a step-by-step approach:

#### Overview
The core idea is to use the ssh command within a GitLab CI job to connect to the remote server and execute deployment tasks. To store the SSH key safely, we will leverage GitLab’s secrets feature.

#### Triage

What is being probed: The interviewer is looking for your understanding of secure practices in CI/CD pipelines and how to implement them using GitLab features.

#### Assumptions
We are working with a GitLab project that has a CI pipeline set up, and the server’s public key is available on the local machine running the pipeline.

#### Solution

Step 1: Securely Store the SSH Key in GitLab Secrets

YAML
variables:
  SSH_PRIVATE_KEY: $SSH_PRIVATE_KEY

In your .gitlab-ci.yml file, you can define a secret variable for the SSH private key.

Step 2: Use the SSH Private Key in Your CI Job

YAML
deploy_to_server:
  stage: deploy
  script:
    - eval `ssh-agent -s`
    - ssh-add <(echo "$SSH_PRIVATE_KEY")
    - ssh user@your.server.com "cd /path/to/deploy; git pull"

In the above script, we start an SSH agent and add the private key to it. Then, we use ssh to connect to the remote server and perform a deployment.

Step 3: Ensure Environmental Security

Environment: Use .env files or environment variables for sensitive data.
Storage: Store secrets outside of version control using GitLab’s secret management features. For example, use the GitLab CI/CD settings to add secrets via the UI.

#### Closing Summary
By securely storing the SSH key as a GitLab secret and leveraging ssh-agent and ssh-add, we can deploy to a server over SSH from a pipeline in a secure manner.

#### Common Mistakes/Follow-Ups

Mistake 1: Storing sensitive keys in plaintext files or hardcoding them into scripts.

#### Follow-Up
Always use GitLab’s secret management features and never expose sensitive information directly in your CI/CD pipeline configuration.
Mistake 2: Failing to clean up after the SSH agent and ssh-add commands.

#### Follow-Up
Ensure that any temporary files or processes related to the SSH key are properly cleaned up.

