---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between SSH and HTTPS when connecting to a GitHub remote?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Git vs GitHub](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_vs_GitHub.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, understanding the differences between SSH (Secure Shell) and HTTPS (HyperText Transfer Protocol Secure) can help you choose the best method based on your needs. TL;DR: SSH is faster and more secure, while HTTPS is easier to set up but less secure.

#### Triaging the Question

The core of this question is about understanding the differences between two protocols used to connect to a GitHub remote repository. The interviewer might be probing your knowledge of Git workflows, security considerations, and network setup skills. We need to clarify that both methods can be used for authentication, but they differ in terms of speed, security, and ease of use.

#### First Principles Solution
SSH (Secure Shell)
Authentication: Uses public-key cryptography, where you generate a pair of keys (public and private) and add the public key to your GitHub account.
Speed: Faster due to direct connection without going through HTTP(S).
Security: Highly secure as it encrypts the entire session from the start to the end.
HTTPS
Authentication: Uses basic authentication or personal access tokens. You need to authenticate each time you interact with the repository over HTTPS.
Speed: Slower compared to SSH because it requires an extra HTTP request for every interaction.
Security: Less secure than SSH as passwords and tokens can be intercepted if not properly secured.
Step-by-Step Solution

Generating Keys:

Use ssh-keygen to generate your public/private key pair on your local machine.

Adding Public Key to GitHub:

Copy the generated public key (id_rsa.pub) to your clipboard and add it under "SSH keys" in your GitHub account settings.

Cloning a Repository via SSH:

BASH
git clone git@github.com:username/repository.git

Working with Repositories via HTTPS:

Clone the repository using https:
BASH
git clone https://github.com/username/repository.git
Use a personal access token (PAT) for authentication when cloning or pushing.

Comparison and Best Practices:

For frequent developers, SSH is preferred due to its speed.
For occasional users or those concerned about security, HTTPS with tokens can be sufficient.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, SSH provides faster and more secure access compared to HTTPS. However, setting up SSH requires generating keys and managing them securely. On the other hand, HTTPS is easier to set up but may introduce latency due to HTTP overhead. Always consider your specific use case when choosing between these methods.

#### Common mistakes
include:

Not understanding the security implications of using passwords with HTTPS.
Failing to properly secure public key files in SSH setup.

For follow-ups, you could discuss additional authentication mechanisms like two-factor authentication (2FA) or how CI/CD systems handle this differently.

**Code:**

```bash
# Example of generating an SSH key pair
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Adding the public key to GitHub
cat ~/.ssh/id_rsa.pub | clip

# Cloning a repository via SSH
git clone git@github.com:username/repository.git

# Cloning a repository via HTTPS with a personal access token
git clone https://<token>@github.com/username/repository.git
```

