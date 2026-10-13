---
type: "practice"
difficulty: "Intermediate"
---

# What is the risk of committing a notebook whose output cell still shows a printed access token or a sample of customer log lines you fed the model?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Notebooks in Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Notebooks_in_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When committing and pushing a Jupyter notebook to a shared repository like GitHub, there is a significant risk if it contains sensitive data such as an access token or sample customer log lines. This exposure can lead to unauthorized access to systems, data breaches, or other security vulnerabilities.

**Code:**

```
Example of code that prints an access token

access_token = "your_secret_token" print(f"Access Token: {access_token}")

CODE
Copy

```python
# Example log line
log_line = '{"customer_id": 12345, "log_level": "INFO", "message": "User logged in"}'
print(log_line)
```

**Example input / output:**

```

Input: A Jupyter notebook with the following code cells:

PYTHON
Copy
access_token = "your_secret_token"
print(f"Access Token: {access_token}")

log_line = '{"customer_id": 12345, "log_level": "INFO", "message": "User logged in"}'
print(log_line)

Output:

CODE
Copy
Access Token: your_secret_token
{"customer_id": 12345, "log_level": "INFO", "message": "User logged in"}
```

**Flow:**

1. Yes
2. No
3. Commit and Push to GitHub
4. Contains Sensitive Data?
5. Security Breach
6. Repository UpdatedSuccessfully

