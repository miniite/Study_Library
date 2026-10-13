---
type: "practice"
difficulty: "Basic"
---

# What is the risk of committing a Jupyter notebook that still contains an output showing an API key or a data sample?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Notebooks in Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Notebooks_in_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you commit a Jupyter notebook that includes outputs such as API keys or sensitive data samples, the risk is significant. These outputs can contain valuable information that could be used maliciously by unauthorized parties to access your data or services.

#### Triage
The core question here is understanding the security risks associated with version control systems like Git and GitHub when they are used improperly. The interviewer is probing for your knowledge of best practices in software development, particularly in handling sensitive information within Jupyter notebooks.

**Code:**

```python
# Example of a code snippet that might include API keys or data samples in output
api_key = "my_secret_api_key"
data_sample = [1, 2, 3, 4, 5]
```

**Example input / output:**

```

Input:

Jupyter notebook with a cell containing the following code snippet:
PYTHON
Copy
import requests

api_key = "my_secret_api_key"
response = requests.get("https://api.example.com/data", headers={"Authorization": f"Bearer {api_key}"})
print(response.json())

Output (uncommitted, in Jupyter runtime):

The API response JSON containing potentially sensitive data.
```

**Data:**

```text
- `api_key`: A string containing sensitive information.
- `data_sample`: A list of integers that might represent sample data points.
```

**Flow:**

1. Yes
2. No
3. Committing a Notebook withSensitive Outputs
4. Is the code visible on GitHub?
5. Data Exposure Risk
6. Safe Commit

