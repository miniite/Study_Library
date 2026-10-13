---
type: "practice"
difficulty: "Basic"
---

# How would you keep secrets like the feature-store DB password out of your Flask ML code and image?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Configuration Management](<../../../../TOOLS/Python/Flask/Configuration_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep secrets such as a feature-store database (DB) password secure in a Flask application, especially when working with machine learning models, we need to separate these sensitive details from our codebase to avoid hardcoding them directly and mitigate the risk of accidental exposure. The key idea is to use environment variables or a configuration management tool to store secrets securely.

#### Triaging the Question

The question is asking how to manage sensitive information like database passwords in a Flask application, ensuring they are not hardcoded into the source code. This involves understanding best practices for handling secrets and leveraging tools that can help secure these credentials without making them part of your version control system or production environment.

#### Solution
from First Principles
Environment Variables: The most straightforward approach is to store sensitive information like database passwords as environment variables. These variables are set in the server's environment rather than hardcoded into your application, reducing the risk of accidental exposure.
Configuration Files: Another method involves storing sensitive data in a configuration file that can be excluded from version control or securely stored separately.
Secret Management Tools: For more complex setups, tools like HashiCorp Vault or AWS Secrets Manager can be used to manage and retrieve secrets programmatically.

#### Implementation
Steps
Set Environment Variables:
Create an .env file in your project root with the necessary environment variables.
PLAINTEXT
FLASK_APP=myapp.py
FEATURE_STORE_DB_PASSWORD=yourpassword
Load these environment variables using a library like python-dotenv.
Use Configuration Files:
Store sensitive data in a YAML or JSON configuration file and load it into your application.
Secret Management Tools:
Integrate with tools like HashiCorp Vault to securely manage and retrieve secrets.

#### Closing Summary

By using environment variables, configuration files, or secret management tools, you can keep sensitive information like database passwords secure in a Flask ML application. This approach ensures that your code remains clean, maintainable, and free from security risks associated with hardcoded secrets.

#### Common Mistakes/Follow-Ups
Hardcoding Secrets: Always avoid hardcoding sensitive information directly into your source code.
Exposure through Environment Variables: Ensure that environment variables are not accidentally committed to version control or exposed in logs.
Security of Configuration Files: Use secure methods to store and manage configuration files, especially when they contain sensitive data.

**Code:**

```python
from dotenv import load_dotenv
import os

# Load environment variables from .env file
load_dotenv()

# Access the database password
db_password = os.getenv('FEATURE_STORE_DB_PASSWORD')
```

**Data:**

```yaml
# example-secrets.yaml
feature_store_db_password: yourpassword
```

