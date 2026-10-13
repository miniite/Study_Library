---
type: "practice"
difficulty: "Deep"
---

# Enforce signed commits on the model-config branch so every change to a safety-critical prompt is attributable. Walk through setting up SSH or GPG signing org-wide, enforcing via branch protection, and what the Verified badge does and does not prove.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Signed Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Signed_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce signed commits on the model-config branch in a safety-critical context, we need to ensure that every change is attributable to an authorized individual. This involves setting up either SSH or GPG signing keys and then configuring GitHub's branch protection rules to require signed commits. The "Verified" badge indicates that the commit has been signed with a key registered with GitHub; however, it does not guarantee the authenticity of the code itself.

#### Triage
What is Being Probed

The question asks for an in-depth understanding of setting up and enforcing signed commits on a specific branch using Git/GitHub. The key aspects being probed include:

SSH vs GPG: Understanding the differences between these two methods.
Org-Wide Setup: How to configure this across multiple repositories or projects.
Branch Protection Rules: Configuring GitHub's settings to enforce signed commits.
"Verified" Badge: What it means and its limitations.

#### Assumptions
The organization uses Git/GitHub for version control.
Developers have the necessary permissions to push changes to the model-config branch.
A key management strategy is in place, ensuring that private keys are securely managed.
Step-by-Step Solution
1. Setting Up SSH or GPG Signing Keys

To set up signing keys:

SH
# Generate a new GPG key (or SSH key if using that)
gpg --gen-key # Follow the prompts to create a new GPG key

# Export the public key for GitHub
gpg --export -a <user_email> > id_rsa.asc

Alternatively, for SSH:

SH
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
2. Adding the Key to GitHub

Upload GPG Key (if using):

PLAINTEXT
# Log in to GitHub
gh auth login

# Add your public key to GitHub
gh config set gpgkey <YOUR_GPG_KEY_ID>

Add SSH Key:

Place the generated id_rsa.pub file into your GitHub account settings under "SSH and GPG keys".
3. Configuring Branch Protection Rules on GitHub
Navigate to the repository in question.
Go to Settings > Branches.
Select the branch you want to protect (model-config).
Under Branch protection rules, enable:
Require status checks: Ensure no new commits can be pushed until all required checks pass.
Enforce admin requirements: Require signed commits.
4. Verifying Signed Commits

The "Verified" badge on GitHub indicates that a commit has been signed with a key registered with your organization’s GPG keys. It does not guarantee the following:

Code Authenticity: The content of the code itself.
Key Ownership: The person who uploaded the key may be different from the actual user.

#### Closing Summary
and Common Mistakes
Common Mistake: Assuming the "Verified" badge guarantees both authenticity and ownership.
Misconfiguration: Not properly configuring branch protection rules, leading to unauthorized changes.
Key Management: Failing to implement a secure key management strategy can lead to vulnerabilities.

#### Follow-Up
Questions
How would you handle cases where keys are compromised or need revocation?
What measures would you take if someone is falsely claiming ownership of commits?

