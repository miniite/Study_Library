---
type: "practice"
difficulty: "Basic"
---

# How do you check which SSH key Git is using when it connects to a remote, and how would you test the connection?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git vs GitHub](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_vs_GitHub.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To check which SSH key Git is using when connecting to a remote repository on GitHub, you can use the ssh -T command or inspect the configuration files manually. For testing the connection, we'll cover both scenarios.

#### Triage
and Assumptions
Objective: Understand the SSH key used for Git connections and test the SSH connectivity.
Tools Involved: Git, SSH.

#### Assumptions
You have set up SSH keys on your local machine.
You are familiar with basic Git commands.
Step-by-Step Solution

Identify the SSH Key in Use

To find out which SSH key is being used by Git for connections to a remote repository, you can use the ssh command along with some flags and options:

SH
ssh -i ~/.ssh/id_rsa git@github.com -T

Here:

-i specifies the path to the private key.
git@github.com is the SSH URL of your repository.

Check Git Configuration

Alternatively, you can check the configuration files where Git stores information about SSH keys:

SH
cat ~/.ssh/config

This file might contain configurations like:

INI
Host github.com
    IdentityFile ~/.ssh/id_rsa

Testing the Connection

To test if the connection works and the correct key is being used, you can push a small commit to your repository or simply clone it again.

SH
git clone git@github.com:your-username/your-repo.git
cd your-repo
echo "Test" > test.txt
git add test.txt
git commit -m "Adding test file"
git push origin main

#### Common Mistakes
and Follow-Ups

Ensure the path to the SSH key is correct in the ~/.ssh/config or command.
Verify that you have added your public key to GitHub under Settings -> SSH and GPG keys.
Check if there are any errors like permission issues or incorrect paths.

#### Closing Summary

By using the ssh command with the -T flag, you can identify which SSH key is being used for Git connections. Checking the ~/.ssh/config file provides additional confirmation. Testing the connection by cloning and pushing to a repository ensures that everything is set up correctly.

#### Common mistakes
include incorrect paths in configuration files or missing public keys on GitHub. Always double-check these details before assuming any issues are with your local environment.

