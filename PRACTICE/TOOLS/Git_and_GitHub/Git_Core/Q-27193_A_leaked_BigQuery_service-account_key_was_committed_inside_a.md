---
type: "practice"
difficulty: "Deep"
---

# A leaked BigQuery service-account key was committed inside a training config, then removed in a follow-up commit. Explain why deleting it in a new commit is useless, and the correct order of rotate, purge history, and notify.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Preventing Secret Leaks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Preventing_Secret_Leaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this situation, we need to understand the implications of committing sensitive data like service-account keys in version control systems such as Git, especially in a cloud environment like Google BigQuery. The key issues here revolve around security best practices, the immutability of Git history, and the importance of proper rotation procedures.

Firstly, when you commit a sensitive file like a service-account key, it gets stored in the repository's history. Even if you remove or rename the file later, its hash remains stored in the commit history. This means that anyone with access to the historical commits can retrieve the file, making the deletion essentially useless from a security perspective.

To mitigate this risk and ensure compliance with best practices, we should follow these steps in the correct order:

Rotate the Key: The first and most critical step is to rotate the key using BigQuery's service-account key management feature. This ensures that any active services using the old key can be updated to use a new, secure key.

Purge History of the Old Key: After ensuring all services are updated with the new key, it’s necessary to remove or obfuscate the history of the old key in Git. This step involves rewriting the commit history and potentially force-pushing changes if other collaborators are also working on the same repository.

Notify Relevant Parties: Once the key is rotated and the history is purged, it's essential to notify all relevant parties (e.g., team members, project leads) about the incident so they can take necessary actions and update their configurations if needed.

**Code:**

```text
# Example of rotating a service-account key using gcloud command
gcloud iam service-accounts keys rotate [SERVICE_ACCOUNT_EMAIL] --key-file=old-key.json

# Example of rewriting Git history (using BFG Repo-Cleaner)
bfg --delete-files old-key.json <repository>.git/
git push origin --force-with-lease
```

**Flow:**

1. Rotate Service Account Key
2. Purge Git History
3. Notify Relevant Parties

