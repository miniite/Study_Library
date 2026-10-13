---
type: "practice"
difficulty: "Deep"
---

# Explain how author and committer diverge on a commit, why the Verified badge attests to the committer's signature not code safety, and how a malicious PR could show a spoofed author name.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Signed Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Signed_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Git and GitHub workflows, the roles of "author" and "committer" can sometimes diverge on a single commit. The author field indicates who originally created the commit, while the committer typically refers to the person who actually pushed or committed the changes to the repository. This distinction is particularly important in scenarios involving code signing or verifying contributions.

GitHub's Verified badge serves as an attestation to the committer’s signature integrity but does not guarantee that the signed code itself is safe from malicious content. The badge merely confirms that the commit was made by a person who has verified their identity via SSH keys, GPG signatures, or other means.

Now, let's dive into how these roles can diverge and why a malicious pull request (PR) could show a spoofed author name:

Understanding Author vs Committer:

The author field is set when the initial commit is created by someone who might be different from the person pushing to the repository.
The committer is the one who pushes the changes and updates the commit metadata, including the date and time.

Why the Verified Badge Only Attests to Identity:

GitHub verifies the committer's identity using various methods like SSH keys or GPG signatures.
This verification process ensures that only the person with the correct credentials can push commits, but it does not check the content of the commit for safety.

Spoofing Author Names in a Malicious PR:

A malicious actor could submit a PR with a spoofed author name by creating an initial commit with a fake author.
They then push this change using their verified credentials, changing the committer information but leaving the author field unchanged.

To illustrate, consider the following scenario:

A user named Alice creates a new branch and writes some code. She wants to contribute it to a project under her username alice.
An attacker has access to Alice's SSH/GPG key and can use this to commit changes.
The attacker forges an initial commit with a spoofed author name, say Bob, but signs the commit using Alice’s verified credentials.

Here is a simplified example:

PLAINTEXT
git commit --author="Bob <bob@example.com>" -m "Malicious code"

In this case, when pushed to GitHub:

The committer would be alice (the attacker using Alice's verified credentials), but the author would still be listed as Bob.
This setup allows the PR to pass initial checks and appear legitimate on the surface.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, while the Verified badge ensures that a commit was made by an identity with appropriate permissions, it does not protect against malicious content. The authorship can be spoofed, making it crucial for reviewers to carefully examine PRs regardless of the verified status.

#### Common mistakes
include assuming that a Verified badge means the code is safe or that the committer's actions align with their identity outside the repository context.

**Code:**

```plaintext
git commit --author="Bob <bob@example.com>" -m "Malicious code"
```

**Example input / output:**

```text
Input:
- Initial commit: `git commit --author="Bob <bob@example.com>" -m "Malicious code"`
- Push to repository with verified credentials

Output:
- Commit metadata shows committer as `alice` but author as `Bob`
- PR appears legitimate on GitHub
```

**Flow:**

1. Signed
2. Unsigned
3. Yes
4. No
5. User creates initial commit
6. Signed or unsigned?
7. Committer pushes changes
8. Committer pushes changes
9. Spoofed author name?
10. PR appears legitimate butauthored by someone else
11. PR looks genuine
12. Review PR for safety andauthenticity

