---
type: "practice"
difficulty: "Basic"
---

# Why should you never hardcode credentials in a Python automation script, and where should they come from instead?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Prompt Injection Defense](<../../../../TOOLS/Python/Python_Language/Prompt_Injection_Defense.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When writing Python scripts for automation, one of the cardinal sins is hardcoding sensitive information such as credentials directly into the codebase. This practice poses significant security risks and can lead to data breaches or unauthorized access. In this response, we'll discuss why hardcoded credentials are a bad idea and explore better alternatives.

Firstly, hardcoded credentials make it easy for anyone with access to your source code to gain unauthorized access to critical systems. If an attacker gains control of the repository or even just views the file, they can extract these credentials and use them maliciously. Secondly, hardcoding means that you must change every instance of the credential whenever a new set is required, such as when rotating keys or changing passwords. This not only wastes time but also introduces human error.

To address these issues, we should leverage externalized secrets management systems. These tools securely store and manage sensitive information, allowing us to reference them from our Python scripts without exposing them directly in the code. Common options include environment variables, configuration files, secret managers like HashiCorp Vault or AWS Secrets Manager, and container orchestration platforms that support secure secrets handling.

Let's take a closer look at how you might implement this using an environment variable as an example:

Store your credentials securely in an environment variable.
Retrieve the value of this environment variable within your Python script using os.environ.
Use the retrieved value for any necessary authentication or API calls.

Here’s a simple code snippet demonstrating this approach:

PYTHON
import os

def fetch_api_data(api_endpoint, username, password):
    # Fetch credentials from environment variables
    if 'API_USERNAME' not in os.environ:
        raise ValueError("Environment variable API_USERNAME is missing")

    user = os.environ['API_USERNAME']
    pwd = os.environ['API_PASSWORD']

    # Now use the fetched credentials to make an API call
    response = requests.get(api_endpoint, auth=(user, pwd))
    return response.json()

# Example usage
api_response = fetch_api_data('https://example.com/api', None, None)
print(api_response)

