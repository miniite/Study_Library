---
type: "practice"
difficulty: "Basic"
---

# What are the basic steps to configure SSH key authentication with GitHub from a Linux server?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Signed Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Signed_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up SSH key-based authentication with GitHub on a Linux server, you need to follow these basic steps:

Generate an SSH key pair if you don't already have one.
Add your public key to your GitHub account.
Test the connection from the server.

Let's break this down step by step using first principles.

**Code:**

```bash
# Step 1: Generate a new SSH key pair (if not done)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Example output:
# Enter a file in which to save the key (/home/you/.ssh/id_rsa):
# Created directory '/home/you/.ssh'.
# Enter passphrase (empty for no passphrase):
# Enter same passphrase again:
```

**Example input / output:**

```text
# Example output after running ssh-keygen
# The key fingerprint is:
# 28:b9:06:5f:d4:3c:e1:7a:5b:a4:cd:6a:3d:ef:ac:ff you@example.com

# The public key content looks like this:
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQdB9tjKxhP+JX...
```

